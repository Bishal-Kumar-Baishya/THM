# Love Letter Locker - CTF Walkthrough

**Target:** 10.49.186.110  
**Difficulty:** Easy  
**Category:** Web Security, IDOR  
**Time to Compromise:** ~15 minutes

---

## Step 1: Reconnaissance

Scan for open ports:

```bash
sudo nmap -sC -sV 10.49.186.110
```

**Results:**
- 22/tcp open ssh
- 5000/tcp open http Werkzeug httpd (Python Flask)


**Finding:** Flask web application running on port 5000

---

## Step 2: Web Application Enumeration

Discover endpoints:
```bash
ffuf -u http://10.49.186.110:5000/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

**Endpoints Found:**
- login [Status: 200]
- register [Status: 200]


---

## Step 3: User Registration

1. Navigate to `/register`
2. Create a test account with any credentials
3. Login with those credentials

---

## Step 4: Application Behavior Analysis

After login:

1. Create a new "love letter" message
2. Submit the letter
3. **Observe the URL:** When viewing your letter, the URL changes to `/letter/3` (or similar number)

**Key Discovery:** 
> "Every love letter gets a unique number in the archive. Numbers make everything easier to find."

This message is the critical hint. Sequential numeric IDs are predictable and enumerable.

---

## Step 5: Vulnerability Identification

**Vulnerability Type:** IDOR (Insecure Direct Object Reference)

**Analysis:**
- Each letter has a sequential numeric ID
- The ID is visible in the URL
- The app doesn't check **WHO** is accessing the letter
- Result: Anyone can access anyone else's letters by guessing/enumerating IDs

**Attack Flow:**
```
Register user A
↓
Create letter → Gets ID (e.g., /letter/3)
↓
Try accessing /letter/1, /letter/2
↓
No authorization check → Access other users' letters
↓
Find flag in someone else's letter
```

---

## Step 6: IDOR Exploitation

### Method 1: Manual Enumeration

Start trying lower letter IDs.

**Results:**
- `/letter/1` ✅ Returns message: **FLAG FOUND**
- `/letter/2` ✅ Returns message: (Other user's letter)

### Method 2: Automated Enumeration (ffuf)

```bash
ffuf -u http://10.49.186.110:5000/letter/FUZZ -w <(seq 1 50)
```

**Output shows:**
- Which letter IDs exist
- Which return 200 (valid)
- Which return 404 (don't exist)

---

## Step 7: Flag Retrieval

Access `/letter/1`:

**Flag:** [REDACTED]

**Explanation:** The first letter in the system contains the flag because there's no authorization check. Anyone can read anyone else's private letters.

---

## Key Vulnerability Details

### Why This is IDOR, Not Just "Guessing"

**IDOR Definition:** Exposing direct references to objects (database records) without verifying the requester has permission.

**In this app:**
```python
# VULNERABLE CODE (likely implementation)
@app.route('/letter/<id>')
def view_letter(id):
    letter = db.get_letter_by_id(id)  # ❌ No permission check
    return render_template('letter.html', letter=letter)

# SAFE CODE
@app.route('/letter/<id>')
def view_letter(id):
    letter = db.get_letter_by_id(id)
    if letter.owner != current_user:  # ✅ Verify ownership
        return "Access Denied", 403
    return render_template('letter.html', letter=letter)
```

### Why Sequential IDs Make It Worse

- **Random IDs:** Attacker must guess (1 in billions) → Harder
- **Sequential IDs:** Attacker simply increments (1, 2, 3...) → Much easier

Sequential: 1, 2, 3, 4, 5, ...
↑ Can predict next ID

Random: a7f3k9, 2b8x1c, q9m2o7, ...
↑ Cannot predict


---

## Attack Impact

**What an attacker can do:**
- ✅ Read any user's private letters
- ✅ Steal personal information
- ✅ Extract secrets/passwords from letters
- ✅ Impersonate users if letters contain sensitive data

**Who is affected:**
- All users of the application
- Not just the attacker's own letters

---

## Mitigation Strategies

### 1. Implement Authorization Checks

```python
@app.route('/letter/<id>')
def view_letter(id):
    letter = db.get_letter_by_id(id)
    
    # CHECK: Does current user own this letter?
    if letter.user_id != current_user.id:
        abort(403)  # Forbidden
    
    return render_template('letter.html', letter=letter)
```

### 2. Use Unpredictable IDs

Instead of sequential (1, 2, 3), use:

```python
import uuid

# Generate unpredictable ID
letter_id = str(uuid.uuid4())  # e.g., "a7f3k9c2-8x1c-4q9m-2o7a-b5c1d2e3f4g5"

# Attacker cannot guess other IDs
# /letter/a7f3k9c2-8x1c-4q9m-2o7a-b5c1d2e3f4g5  ✅ Safe
# /letter/1  ❌ Doesn't exist
```

### 3. Proper Database Queries

```python
# UNSAFE: No user filter
letter = Letter.query.filter_by(id=id).first()

# SAFE: Filter by both ID and owner
letter = Letter.query.filter_by(
    id=id, 
    user_id=current_user.id
).first()

if not letter:
    abort(404)
```

---

## Real-World IDOR Examples

| Service | Vulnerable Endpoint | Impact |
|---------|-------------------|--------|
| Banking App | `/account/1234567` | Read other account balances |
| Social Media | `/profile/userid` | View private profiles |
| E-commerce | `/order/5678` | Read other customer orders |
| Healthcare | `/patient/record/123` | Access medical records |
| Government | `/document/id/456` | Access classified files |

All of these would be IDOR if there's no ownership verification.

---

## OWASP Top 10 Mapping

- **A01:2021 – Broken Access Control**
  - Lack of authorization on resources
  - Sequential/predictable ID disclosure

---

## Summary

| Phase | Action | Result |
|-------|--------|--------|
| Reconnaissance | nmap scan | Found Flask app on 5000 |
| Enumeration | ffuf discovery | Found /login and /register |
| Registration | Create account | Gained authenticated access |
| Analysis | Create letter | Noticed sequential ID in URL |
| Exploitation | Try /letter/1 | Accessed other user's letter |
| Flag | Read /letter/1 | Retrieved flag |

**Total Time:** ~15 minutes  
**Difficulty:** Easy  
**Learning Value:** Critical (IDOR fundamentals)

