# Signed Messages - CTF Challenge Write-Up

## Overview

**Signed Messages** is a medium-difficulty TryHackMe CTF challenge that demonstrates how cryptographic vulnerabilities, combined with weak authentication, lead to complete system compromise.

**Challenge Link:** [TryHackMe - Signed Messages](https://tryhackme.com/room/signedmessages)  
**Difficulty:** Medium  
**Category:** Web Security, Cryptography  
**Time to Solve:** ~1.5 hours

---

## What You'll Learn

This challenge teaches critical security concepts through hands-on exploitation:

### Core Vulnerabilities Exploited

1. **Passwordless Authentication** — Login with only a username
2. **Information Disclosure** — Debug endpoint exposes key generation algorithm
3. **Predictable Cryptography** — RSA seeds based on public information
4. **Signature Forgery** — Reconstruct private keys and forge admin signatures

### Security Concepts Covered

- **RSA Cryptography** — How digital signatures work and why randomness matters
- **Secure Key Generation** — Difference between predictable and truly random seeds
- **Defense in Depth** — Why multiple security layers are necessary
- **OWASP Top 10** — Information Exposure, Broken Authentication, Cryptographic Failures

---

## Challenge Description

You're tasked with compromising a "Love Note" web application that uses digital signatures to verify message authenticity. The app claims to be secure because it uses RSA signatures, but poor implementation creates exploitable weaknesses.

**Your Goal:** Forge an admin signature to capture the flag.

**What You Get:**
- Target IP address and port
- Access to a Flask web application
- Several endpoints (login, compose, verify, debug)

**What You Need to Find:**
- How to authenticate without a password
- Where the cryptographic secrets are exposed
- How to reconstruct the admin's private key
- How to forge a valid signature

---

## Challenge Architecture

### Vulnerable Components# Signed Messages - CTF Challenge Write-Up

## Overview

**Signed Messages** is a medium-difficulty TryHackMe CTF challenge that demonstrates how cryptographic vulnerabilities, combined with weak authentication, lead to complete system compromise.

**Challenge Link:** TryHackMe - Signed Messages
**Difficulty:** Medium  
**Category:** Web Security, Cryptography  
**Time to Solve:** ~1.5 hours

---

## What You'll Learn

This challenge teaches critical security concepts through hands-on exploitation:

### Core Vulnerabilities Exploited

1. **Passwordless Authentication** — Login with only a username
2. **Information Disclosure** — Debug endpoint exposes key generation algorithm
3. **Predictable Cryptography** — RSA seeds based on public information
4. **Signature Forgery** — Reconstruct private keys and forge admin signatures

### Security Concepts Covered

- **RSA Cryptography** — How digital signatures work and why randomness matters
- **Secure Key Generation** — Difference between predictable and truly random seeds
- **Defense in Depth** — Why multiple security layers are necessary
- **OWASP Top 10** — Information Exposure, Broken Authentication, Cryptographic Failures

---

## Challenge Description

You're tasked with compromising a "Love Note" web application that uses digital signatures to verify message authenticity. The app claims to be secure because it uses RSA signatures, but poor implementation creates exploitable weaknesses.

**Your Goal:** Forge an admin signature to capture the flag.

**What You Get:**
- Target IP address and port
- Access to a Flask web application
- Several endpoints (login, compose, verify, debug)

**What You Need to Find:**
- How to authenticate without a password
- Where the cryptographic secrets are exposed
- How to reconstruct the admin's private key
- How to forge a valid signature

---

## Challenge Architecture

### Application Endpoints

| Endpoint | Auth Required | Purpose | Vulnerability |
|----------|---------------|---------|----------------|
| `/login` | No | Login form | ❌ No password field |
| `/register` | No | Account creation | User enumeration possible |
| `/dashboard` | Yes | User dashboard | Accessible if auth bypassed |
| `/debug` | **No** | Key generation info | ❌ **Information leak** |
| `/verify` | No | Signature verification | Accepts forged signatures |
| `/compose` | Yes | Message creation | N/A (not exploited) |


---

## Quick Start

### Prerequisites

**System Requirements:**
- Linux/macOS (or WSL on Windows)
- Python 3.8+
- Internet connection to TryHackMe

**Required Tools:**
```bash
# Install required packages
pip install cryptography sympy

# Install system utilities (if not already installed)
sudo apt install nmap ffuf  # Ubuntu/Debian
brew install nmap ffuf      # macOS
```

### Running the Exploit

1. **Access the TryHackMe room** and start the target machine
2. **Get your target IP** (e.g., `10.48.160.162`)
3. **Run reconnaissance:**
```bash
   nmap -sC -sV 10.48.160.162
```

4. **Execute the exploit:**
```bash
   python3 exploit.py
```
   Input: `admin`

5. **Visit the verification endpoint:**
   - Navigate to `http://10.48.160.162:5000/verify`
   - Submit the forged signature
   - Capture the flag


---

## Detailed Exploitation Guide

### For Beginners

If you're new to security:

1. **Start with WALKTHROUGH.md** — Follow step-by-step instructions
2. **Understand each phase** before moving to the next
3. **Run commands exactly as shown** to avoid confusion
4. **Take notes** on what each vulnerability does
5. **Research** the security concepts (RSA, hashing, signatures)

**Recommended Reading:**
- [RSA Cryptography Basics](https://en.wikipedia.org/wiki/RSA_(cryptosystem))
- [Digital Signatures Explained](https://en.wikipedia.org/wiki/Digital_signature)
- [OWASP: Information Exposure](https://owasp.org/www-project-top-ten/)

### For Intermediate Learners

If you have security experience:

1. **Review the exploit.py script** — Understand the key reconstruction logic
2. **Trace the attack chain** — How do vulnerabilities chain together?
3. **Modify the exploit** — What if the seed pattern was different?
4. **Test mitigations** — What if the app required a password?
5. **Write your own version** — Implement the exploit from scratch

**Challenge Questions:**
- What would happen if the seed included random data?
- How would the attack change if `/debug` was protected by authentication?
- Can you exploit this with a different message instead of "I am the admin"?

### For Advanced Practitioners

If you're experienced in penetration testing:

1. **Analyze security gaps** — Use PENTEST_REPORT.md as a reference
2. **Review the architecture** — How would you redesign this application?
3. **Consider defense strategies** — What mitigations would be most effective?
4. **Extend the exploit** — Can you automate the entire attack chain?
5. **Document findings** — Write your own security assessment

**Advanced Topics:**
- Implementing secure key derivation (PBKDF2, Argon2)
- Hardware Security Module (HSM) integration
- Cryptographic agility and algorithm rotation
- Secure development lifecycle (SDLC) practices

---

## Key Vulnerabilities

### 1. Passwordless Authentication (CRITICAL)

**CVSS Score:** 9.8 (Critical)

**Description:**
The login form accepts only a username with no password field. Any attacker can impersonate any user, including the admin.

**Impact:**
- Unauthorized access to user accounts
- No accountability (anyone can pose as anyone)
- Breaks the entire security model

**Root Cause:**
- Incomplete authentication implementation
- No identity verification mechanism

**Mitigation:**
```python
# WRONG (current implementation)
@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    session['user'] = username  # ❌ No password check

# CORRECT (with password verification)
@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    password = request.form.get('password')
    
    # Verify password against database hash
    user = User.query.filter_by(username=username).first()
    if user and verify_password(password, user.password_hash):
        session['user'] = username
        return redirect('/dashboard')
    else:
        return "Invalid credentials", 401
```

### 2. Predictable Cryptographic Seeds (CRITICAL)

**CVSS Score:** 9.8 (Critical)

**Description:**
RSA private keys are generated using a predictable seed based only on the username: `seed = f"{username}_lovenote_2026_valentine"`


**Impact:**
- Attacker can reconstruct ANY user's private key offline
- No need to access server's key storage
- Digital signatures become forgeable

**Root Cause:**
- Using public information (username) in key generation
- No cryptographic randomness in seed

**Mitigation:**
```python
# WRONG (current implementation)
def generate_keys(username):
    seed = f"{username}_lovenote_2026_valentine"  # ❌ Predictable
    random_seed = hashlib.sha256(seed.encode()).digest()
    # ... generate keys from seed

# CORRECT (using secure randomness)
import os
from cryptography.hazmat.primitives.asymmetric import rsa

def generate_keys(username):
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=2048,
        backend=default_backend()
    )
    # Keys are generated using os.urandom() internally
    return private_key
```

### 3. Information Disclosure via Debug Endpoint (HIGH)

**CVSS Score:** 7.5 (High)

**Description:**
The `/debug` endpoint is publicly accessible and reveals:
- RSA key generation algorithm
- Seed pattern
- Internal architecture details

**Impact:**
- Attackers learn the exact attack vector
- Enables rapid exploitation
- "Security through obscurity" is broken

**Root Cause:**
- Debug endpoint left in production
- No authentication on debugging features

**Mitigation:**
```python
# WRONG (current implementation)
@app.route('/debug')
def debug():
    return render_template('debug.html', seed_pattern=seed_pattern)  # ❌ Public

# CORRECT (disable in production)
import os

@app.route('/debug')
def debug():
    if os.getenv('FLASK_ENV') == 'production':
        abort(404)  # Hide debug endpoint
    return render_template('debug.html', seed_pattern=seed_pattern)

# BETTER (require authentication)
@app.route('/debug')
@admin_required  # Authentication decorator
def debug():
    return render_template('debug.html', seed_pattern=seed_pattern)
```

---

## Security Lessons Learned

### 1. Defense in Depth

**The Problem:**
The app relied on signatures being "secure" without proper authentication.

**The Lesson:**
Security is layered. Even if one layer (signatures) is mathematically correct, other layers (authentication, key generation) can fail.

**Apply This:**
- Require authentication AND authorization
- Implement multiple verification mechanisms
- Don't assume one security control is enough

### 2. Cryptography is Hard

**The Problem:**
The developers used RSA correctly for signature verification, but failed at key generation.

**The Lesson:**
Cryptography libraries make signing easy, but security depends on how you generate and protect keys.

**Apply This:**
- Use libraries' built-in randomness functions
- Never create your own RNG
- Generate keys separately from the app logic
- Store keys securely (HSM, sealed storage, etc.)

### 3. Debug Information is Dangerous

**The Problem:**
Debug logs in production revealed the attack vector.

**The Lesson:**
Developers often help attackers by documenting their own systems.

**Apply This:**
- Remove debug endpoints in production
- Never log sensitive information (keys, seeds, algorithms)
- Use environment-based configuration
- Treat "what attackers can learn" as a threat model

### 4. Secure Defaults Matter

**The Problem:**
The app didn't enforce passwords or randomness.

**The Lesson:**
It's easy to forget about security when building features quickly.

**Apply This:**
- Use frameworks' built-in security features
- Make "secure by default" the standard
- Review security checklist before deployment
- Conduct threat modeling during design phase

---

## How to Use This Write-Up

### For Learning

1. **Attempt the challenge first** — Don't read the walkthrough immediately
2. **When stuck, read step-by-step** — Use hints, not full solutions
3. **Understand the "why"** — Why does each vulnerability exist?
4. **Experiment** — Try variations (different usernames, messages, etc.)

### For Teaching Others

1. **Share WALKTHROUGH.md** — Show the methodology, not just commands
2. **Discuss vulnerabilities** — Explain the security impact
3. **Walk through the code** — Show exploit.py and how it works
4. **Ask questions** — "What if we changed X? Would it still work?"

### For Your Portfolio

1. **Include this README** — Shows you understand the context
2. **Add PENTEST_REPORT.md** — Demonstrates professional assessment skills
3. **Reference security standards** — OWASP, CVSS scores, etc.
4. **Show remediation** — Prove you can fix vulnerabilities, not just exploit them

---

## Resources & Further Reading

### Cryptography Concepts

- **RSA Cryptography:** [Wikipedia - RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem))
- **Digital Signatures:** [OWASP - Digital Signatures](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- **Secure Key Generation:** [NIST SP 800-133 - Key Derivation](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-133r1.pdf)

### Web Security

- **OWASP Top 10:** [www.owasp.org/www-project-top-ten](https://owasp.org/www-project-top-ten/)
- **Authentication Bypass:** [OWASP - Broken Authentication](https://owasp.org/www-community/attacks/authentication_cheat_sheet)
- **Information Disclosure:** [CWE-200: Information Exposure](https://cwe.mitre.org/data/definitions/200.html)

### Python & Cryptography

- **Cryptography Library:** [cryptography.io](https://cryptography.io/)
- **Python Security:** [Real Python - Cryptography](https://realpython.com/python-cryptography/)

### CTF & Penetration Testing

- **HackTheBox Academy:** [academy.hackthebox.com](https://academy.hackthebox.com/)
- **TryHackMe:** [tryhackme.com](https://tryhackme.com/)
- **OverTheWire Wargames:** [overthewire.org](https://overthewire.org/)

---

## Common Issues & Troubleshooting

### Issue: "Command not found: nmap"

**Solution:**
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install nmap

# macOS
brew install nmap

# Verify installation
nmap --version
```

### Issue: "ModuleNotFoundError: No module named 'cryptography'"

**Solution:**
```bash
pip install cryptography sympy
# or
pip3 install cryptography sympy
```

### Issue: "Connection refused" when accessing http://10.48.160.162:5000

**Solution:**
- Make sure TryHackMe machine is running (green indicator in room)
- Verify you're on the correct IP (copy from room interface)
- Check if your VPN connection is active
- Try: `ping 10.48.160.162` to test connectivity

### Issue: Signature verification fails on /verify endpoint

**Solution:**
- Ensure you copied the entire hex signature (no spaces/truncation)
- Verify sender is set to `admin` (exact case)
- Verify message is `I am the admin` (exact text)
- Re-run exploit.py with correct username

### Issue: Script runs but generates different signature each time

**This is expected!** RSA-PSS padding includes randomness. Each signature is unique even for the same message, but all are valid. The server will accept any of them.

---

## Disclaimer & Legal Notice

**Educational Purpose Only:**

This write-up is created for educational purposes on authorized TryHackMe machines. 

**Do NOT:**
- Use these techniques on systems you don't own or have permission to test
- Exploit vulnerabilities in production systems
- Attempt to compromise real applications without authorization

**Do:**
- Use this knowledge on dedicated lab environments (HTB, TryHackMe, DVWA, etc.)
- Participate in legal bug bounty programs
- Only test systems you own or have written permission to test

---

## Author

**Bishal Kumar Baishya** — Cybersecurity Student | Penetration Testing Focus

---

## License

This write-up is provided as-is for educational purposes. Feel free to use, modify, and share with proper attribution.

---

**Last Updated:** October 2, 2026

**Questions?** Review the WALKTHROUGH.md for step-by-step guidance or check the Troubleshooting section above.