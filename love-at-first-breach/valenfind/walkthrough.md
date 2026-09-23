# Love at first breach - ValenFind CTF - Complete Walkthrough

**Target IP:** 10.49.179.188<br>
**Difficulty:** Medium

## 1. Reconnaissance

### Network Scanning
```bash
nmap -sC -sV 10.49.179.188
```

**Results:**
- Port 22: SSH
- Port 5000: HTTP (Flask Web Application)


### Application Discovery
- Visited `http://10.49.179.188:5000`
- Registered and logged in
- Explored user profiles and dashboard


## 2. Vulnerability Analysis

### Initial Attempts (Dead Ends)
- Tested for IDOR on profiles
- Attempted SQL injection on login
- Used gobuster for endpoint enumeration
- Inspected source code for hardcoded secrets

### Breakthrough: Source Code Analysis
While inspecting network requests, discovered suspicious JavaScript code:

```javascript
fetch(`/api/fetch_layout?layout=${layoutName}`)
    .then(r => r.text())
    .then(html => {
        // Client-side rendering of fetched template
        let rendered = html.replace('__USERNAME__', username)
                           .replace('__BIO__', bioText);
        document.getElementById('bio-container').innerHTML = rendered;
    })
```
**Issue Identified:** User input directly concatenated into file path.


## 3. Exploitation

### Step 1: Read System Files
```
GET /api/fetch_layout?layout=../../../../etc/passwd
```

Successfully read `/etc/passwd` revealing system users.

### Step 2: Discover Application Path
From error message: [Errno 21] Is a directory: '/opt/Valenfind/templates/components/...'

**Application located at:** `/opt/Valenfind/`

### Step 3: Extract Source Code
```
GET /api/fetch_layout?layout=../../app.py
```

Retrieved full Flask source code, revealing:
- `ADMIN_API_KEY = "CUPID_MASTER_KEY_2024_XOXO"`
- Database structure
- Admin endpoint at `/api/admin/export_db`


### Step 4: Download Database
Using **Burp Suite**, added header: X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO

Downloaded entire SQLite database.

### Step 5: Extract Flag
Opened database in SQLite Browser, browsed users table.

Found flag in the "cupid" user's bio field.


## 4. Key Learnings
- Error messages reveal infrastructure paths
- Always check source code when tools fail
- Hardcoded credentials are fatal
- Single vulnerability can cascade to full compromise 

