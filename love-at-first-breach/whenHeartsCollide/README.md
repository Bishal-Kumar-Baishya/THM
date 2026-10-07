# When hearts collide - CTF

## What Was This Challenge?

A web application that matches dog photos using MD5 hashing. The vulnerability: **MD5 collisions**.

---

## The Vulnerability in 30 Seconds

```
App checks:
1. Does uploaded file's MD5 match a dog in database?
2. Is uploaded file NOT an exact duplicate?

Both checks rely on MD5 (broken algorithm)

Attack:
- Create two files with SAME MD5 but DIFFERENT content
- Upload collision file
- Passes check 1: MD5 matches ✅
- Passes check 2: Content differs ✅
- 🚩 FLAG OBTAINED
```

---

## Quick Exploit

### 1. Install fastcoll
```bash
sudo apt install build-essential libboost-all-dev libboost-filesystem-dev libboost-program-options-dev
git clone https://github.com/upbit/clone-fastcoll.git
cd clone-fastcoll
g++ -O3 *.cpp -o fastcoll -lboost_filesystem -lboost_program_options
```

### 2. Download Featured Dog
- Visit application homepage
- Find featured dog image
- Download it

### 3. Generate Collision
```bash
./fastcoll /path/to/dog.jpg
md5sum md5_data*  # Verify both have same hash
```

### 4. Upload Collision File
```bash
curl -X POST http://target/upload -F "file=@md5_data1"
```

### 5. Get Flag
Follow redirect to success page and retrieve flag.

---

## Key Concepts

### MD5 Hash Collision
Two different files with identical MD5 hash value.

```
File A (original dog):    binary content XYZ...   → MD5: abc123
File B (collision):       binary content ABC...   → MD5: abc123
```

### Why It Works
- App compares hashes → sees match ✅
- App compares binary → sees difference ✅
- Both checks pass, despite File B being completely different

### Why MD5 is Weak
- Cryptographically broken since 2004
- Collision attacks practical with modern tools
- Never use for security

---

## Tools Reference

| Command | Purpose |
|---------|---------|
| `nmap -sC -sV [ip]` | Discover services |
| `ffuf -u http://[ip]/FUZZ -w [wordlist]` | Find directories |
| `curl -X POST [url] -F "file=@[file]"` | Upload file |
| `md5sum [file]` | Calculate MD5 hash |
| `./fastcoll [input]` | Generate MD5 collision |

---

## Reconnaissance Findings

```
Endpoints:
- GET  /                    → Homepage with featured dog
- POST /upload              → File upload endpoint
- GET  /static/uploads/[UUID].jpg  → Access uploaded files
- GET  /view/[UUID]         → View upload result

File Structure:
/static/uploads/[UUID].jpg  → Uploaded dog images stored here
```

---

## Attack Flow

```
1. Reconnaissance
   └─ Identify app uses MD5 hashing
   └─ Find upload endpoint
   └─ Locate featured dog image

2. Initial Analysis
   └─ Download featured dog
   └─ Attempt direct upload → FAILS (duplicate detected)
   └─ Confirms duplicate detection exists

3. Vulnerability Exploitation
   └─ Research MD5 weaknesses
   └─ Install fastcoll tool
   └─ Generate collision files
   └─ Upload collision file → SUCCEEDS
   └─ Retrieve flag
```

---

## Why This Worked

| Stage | Check | Result |
|-------|-------|--------|
| Upload collision.jpg | - | START |
| Calculate MD5 | Same as database dog | ✅ PASS |
| Check duplicate? | Binary differs from dog | ✅ PASS |
| Both pass? | YES | ✅ FLAG |

---

## Common Mistakes (Avoid)

❌ **Wrong:** Try to brute-force MD5 offline  
✅ **Right:** Use fastcoll for practical collisions

❌ **Wrong:** Try to modify image to keep same hash  
✅ **Right:** Generate collision with different tool output

❌ **Wrong:** Upload original dog again  
✅ **Right:** Upload collision file (different binary, same hash)

---

## Security Takeaway

### Never Use MD5 For Security

**Bad:**
```python
if hashlib.md5(file).hexdigest() == database_hash:
    allow_upload()
```

**Good:**
```python
if hashlib.sha256(file).hexdigest() == database_hash:
    allow_upload()
```

### Defense in Depth

Don't rely on single algorithm. Combine:
- Cryptographic hash (SHA-256)
- File type validation (MIME, magic bytes)
- Content analysis (image structure)
- Duplicate detection (independent mechanism)

---

## Flag Location

After successful collision upload:
- Check `/upload_success/[UUID]` redirect
- Or check application response
- Flag displayed on success page

---

## Time to Complete

- Reconnaissance: 10-15 minutes
- Vulnerability analysis: 5-10 minutes
- Tool setup: 5-10 minutes
- Exploitation: 2-3 minutes
- **Total: ~30 minutes**

---

## Further Reading

- [MD5 Cryptanalysis](https://en.wikipedia.org/wiki/MD5#Vulnerabilities)
- [Practical MD5 Collisions](https://hashclash.github.io/)
- [OWASP Hash Algorithms](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)

---

**Status:** ✅ Completed | **Difficulty:** Medium | **Type:** Web + Crypto