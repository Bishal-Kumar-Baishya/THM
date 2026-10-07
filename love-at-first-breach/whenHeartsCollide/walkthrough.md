# When hearts collide CTF - Complete Writeup

## Challenge Overview

**Platform:** TryHackMe  
**Category:** Web Security + Cryptography  
**Difficulty:** Medium  
**Objective:** Exploit a dog matching application that uses MD5 hashing

---

## Application Architecture

### What is DogMatcher?
A web application that matches user-uploaded dog photos against a database of curated dogs using MD5 hash comparison.

**Flow:**
1. User uploads a photo
2. App calculates MD5 hash of upload
3. Compares against database of curated dog hashes
4. If match → Success message
5. If duplicate → "Already uploaded" message

### The Quote That Reveals Everything
> "When two MD5s lock eyes, the lights flare, the wagging intensifies, and the screen happily announces: you are matched."

This hints at the entire vulnerability: the app relies solely on MD5 hashing.

---

## Reconnaissance Phase

### 1. Service Discovery
```bash
nmap -sC -sV 10.49.187.70
```

**Results:**
- Port 22: SSH (OpenSSH 9.6p1)
- Port 80: HTTP (nginx)

### 2. Directory Enumeration
```bash
ffuf -u http://10.49.187.70/FUZZ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```

**Key Findings:**
- `/static` [301] - Static assets directory
- `/upload` [405] - Upload endpoint (Method Not Allowed on GET)

### 3. Static Directory Structure
```bash
ffuf -u http://10.49.187.70/static/FUZZ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```

**Discovered:**
- `/static/fonts` [301]
- `/static/uploads` [301] - User uploads stored here
- `/static/vendor` [301]

### 4. Access Control Test
```bash
curl http://10.49.187.70/static/uploads/
# Returns: 403 Forbidden
```

Directory listing is disabled (nginx protection).

---

## Application Analysis

### Homepage Inspection
The homepage features:
- A random/featured dog image
- Upload section for user photos
- Database section showing curated dogs
- Clear indication that matching uses MD5

### Upload Endpoint Behavior

**Test 1: Upload with GET (Wrong Method)**
```bash
curl http://10.49.187.70/upload
# 405 Method Not Allowed
```

**Test 2: Upload with POST (Correct Method)**
```bash
curl -X POST http://10.49.187.70/upload -F "file=@/home/light/Downloads/dog.jpg"
```

**Response:**
```html
<p>You should be redirected automatically to the target URL: 
<a href="/upload_success/50313741-c398-4fa6-9fc7-e64953b211c2">
/upload_success/50313741-c398-4fa6-9fc7-e64953b211c2</a>
```

### Success Page Analysis
```bash
curl http://10.49.187.70/upload_success/50313741-c398-4fa6-9fc7-e64953b211c2
```

Response shows:
- "Your photo already lives here"
- "We already received that exact snapshot"
- "our hash-powered memory is working hard and just marked your upload as a repeat"

**Implication:** The uploaded dog already exists in the database → automatic match!

### Page Source Examination

When viewing featured dog via `/view/[UUID]`:
```html
<!DOCTYPE html>
<html>
<head><title>Matchmaker</title></head>
<body>
  <h1>This could be your dog</h1>
  <img src="/static/uploads/00795a8b-fb58-47c0-91be-af068ddc71b4.jpg" 
       alt="Uploaded profile">
</body>
</html>
```

**Discovery:** Direct file path reveals UUID-based storage system.

---

## Vulnerability Analysis

### The Two-Layer Protection (That Failed)

The app implements:
1. **MD5 Hash Matching:** Compare upload hash to database
2. **Duplicate Detection:** Reject if binary content is identical

**Check 1: Hash Verification**
```
if upload_md5 == database_dog_md5:
    return MATCH
```

**Check 2: Duplicate Check**
```
if upload_binary == database_dog_binary:
    return DUPLICATE_ERROR
```

### Why Direct Upload Failed

```bash
md5sum ./Downloads/dog.jpg
# a15ec1ecaef0eac2d8a9be79d1d51296

# Upload attempt
curl -X POST http://10.49.187.70/upload -F "file=@dog.jpg"
# Result: "Your photo already lives here"
```

The uploaded dog:
- ✅ Passed MD5 check (matches database)
- ❌ Failed duplicate check (exact same binary content)

### The Exploit: MD5 Collision Attack

**Vulnerability:** Both checks rely on MD5, which is **cryptographically broken**.

MD5 allows generating two **different files** with the **same hash** (collision attack).

**If we create a collision:**
- File A: Original dog (binary content)
- File B: Different bytes, but same MD5 as File A

Then:
- ✅ Check 1 passes: MD5 matches database
- ✅ Check 2 passes: Binary content is different (not duplicate)
- 🚩 **Vulnerability bypassed!**

---

## Exploitation Steps

### Step 1: Set Up Collision Tool

```bash
# Install dependencies
sudo apt install build-essential libboost-all-dev git libboost-filesystem-dev libboost-program-options-dev

# Clone fastcoll
git clone https://github.com/upbit/clone-fastcoll.git
cd clone-fastcoll

# Compile
g++ -O3 *.cpp -o fastcoll -lboost_filesystem -lboost_program_options
```

### Step 2: Identify Target Dog

The featured dog on the homepage:
```bash
md5sum ./Downloads/dog.jpg
# a15ec1ecaef0eac2d8a9be79d1d51296
```

This hash is in the database (we know because direct upload was flagged as duplicate).

### Step 3: Generate MD5 Collision

```bash
./fastcoll /home/light/Pictures/dog.jpg
```

**Process:**
- Tool exploits known MD5 weaknesses
- Generates two files with identical MD5
- Files are completely different in binary content

**Output:**
```
Generating first block: ...............
Generating second block: S11...
use 'md5sum md5_data*' check MD5

md5sum md5_data*
# 2ef802b7964c635e1c7113a70293f516  md5_data1
# 2ef802b7964c635e1c7113a70293f516  md5_data2
```

**Verification:** Both files have identical MD5, but different content.

### Step 4: Upload Collision File

```bash
mv md5_data1 collision.jpg

curl -X POST http://10.49.187.70/upload -F "file=@collision.jpg"
```

**Result:**
```html
<a href="/upload_success/[NEW-UUID]">Success!</a>
```

**Why it works:**
1. **MD5 Check:** collision.jpg has same MD5 as database dog ✅
2. **Duplicate Check:** collision.jpg binary content differs from original dog ✅
3. **Both checks pass** → System thinks it's a NEW dog that matches!

### Step 5: Retrieve Flag

Access the success page or check the application's response for the flag.

---

## Vulnerability Chain

```
┌─────────────────────────────────────────────┐
│  User uploads dog image from homepage       │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│  Calculate MD5 hash of upload               │
│  Example: a15ec1ecaef0eac2d8a9be79d1d51296  │
└────────────┬────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────┐
│  Check 1: MD5 matches database?      │
│  ✅ YES (dog is in database)         │
└────────────┬─────────────────────────┘
             │
             ▼
┌──────────────────────────────────────┐
│  Check 2: Exact duplicate?           │
│  ❌ NO (binary content differs)      │
│  (if using collision file)           │
└────────────┬─────────────────────────┘
             │
             ▼
┌──────────────────────────────────────┐
│  Both checks pass!                   │
│  🚩 FLAG OBTAINED                    │
└──────────────────────────────────────┘
```

---

## Technical Deep Dive: MD5 Collisions

### Why MD5 is Broken

1. **Collision vulnerabilities discovered:** 2004-2005
2. **Practical attacks exist:** fastcoll, HashClash tools
3. **Computational complexity:** Feasible with modern hardware

### How fastcoll Works

1. Takes an input file as "template"
2. Appends collision-generating bytes
3. Creates second file with same hash but different content
4. Exploits known MD5 weaknesses in specific message blocks

### Why This Defeats the App

| Aspect | Original Dog | Collision File | Result |
|--------|-------------|----------------|--------|
| MD5 Hash | `a15ec1ec...` | `a15ec1ec...` | ✅ Identical |
| Binary Content | Original bytes | Modified bytes | ✅ Different |
| Duplicate Check | Triggers | Bypasses | ✅ Bypassed |
| Match Check | Required | Matched | ✅ Passed |

**Result:** App treats collision file as valid new dog that matches!

---

## Key Security Lessons

### 1. Never Use MD5 for Security
```python
# ❌ WRONG
file_hash = hashlib.md5(file_content).hexdigest()
if file_hash == database_hash:
    mark_as_match()

# ✅ CORRECT
file_hash = hashlib.sha256(file_content).hexdigest()
if file_hash == database_hash:
    mark_as_match()
```

### 2. Defense in Depth
- Don't rely on single algorithm for multiple security decisions
- Combine hash verification with:
  - File type validation (MIME type)
  - Magic byte verification
  - Content-based analysis

### 3. Proper File Validation
```python
# Better approach
def is_valid_dog_image(file_content):
    # Check magic bytes
    if not file_content.startswith(b'\xFF\xD8\xFF'):  # JPEG signature
        return False
    
    # Check file structure
    try:
        Image.open(BytesIO(file_content))
        return True
    except:
        return False

def verify_match(upload, database):
    # Use SHA-256 for matching
    upload_hash = hashlib.sha256(upload).hexdigest()
    database_hash = hashlib.sha256(database).hexdigest()
    return upload_hash == database_hash
```

### 4. Remediation Checklist
- [ ] Replace MD5 with SHA-256 or SHA-3
- [ ] Implement independent duplicate detection
- [ ] Add file type validation
- [ ] Use magic byte verification
- [ ] Consider content-addressable storage (content hash)
- [ ] Log collision attempts for security monitoring

---

## Tools Used

| Tool | Purpose | Command |
|------|---------|---------|
| **nmap** | Service discovery | `nmap -sC -sV [target]` |
| **ffuf** | Directory enumeration | `ffuf -u [url]/FUZZ -w [wordlist]` |
| **curl** | HTTP requests | `curl -X POST [url] -F "file=@[file]"` |
| **md5sum** | Hash calculation | `md5sum [file]` |
| **fastcoll** | MD5 collision generation | `./fastcoll [input-file]` |
| **git** | Clone fastcoll repo | `git clone [repo]` |
| **g++** | Compile fastcoll | `g++ -O3 *.cpp -o fastcoll` |

---

## Timeline

| Step | Action | Result |
|------|--------|--------|
| 1 | Reconnaissance (nmap, ffuf) | Identified web app and endpoints |
| 2 | Homepage analysis | Found featured dog image |
| 3 | Direct upload attempt | Failed (duplicate detection) |
| 4 | Vulnerability analysis | Identified MD5 weakness |
| 5 | Install fastcoll | Set up collision generation tool |
| 6 | Generate collision | Created two files with same MD5 |
| 7 | Upload collision file | Bypassed both checks |
| 8 | Flag retrieval | ✅ **Challenge completed** |

---

## Conclusion

**DogMatcher** demonstrates the critical importance of using cryptographically sound algorithms for security decisions. By relying on MD5 for both matching and duplicate detection, the application created a vulnerability that could be exploited through practical hash collision attacks.

**Key Takeaway:** Never use MD5 for security. SHA-256 and SHA-3 remain secure alternatives for the foreseeable future.

---

## References

- [MD5 Vulnerabilities](https://en.wikipedia.org/wiki/MD5#Vulnerabilities)
- [fastcoll GitHub](https://github.com/upbit/clone-fastcoll)
- [OWASP: Hash Functions](https://owasp.org/www-community/attacks/Hash_collisions)
- [Cryptographic Hashing Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)

---

**Challenge Status:** ✅ COMPLETED