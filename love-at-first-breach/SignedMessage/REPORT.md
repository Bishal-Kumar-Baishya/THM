# Penetration Testing Report
## Signed Messages - CTF

---

## Document Information
- **Report Title:** Penetration Testing Report - Signed Message
- **Date:** 01-10-2026
- **Target:** 10.48.160.162
- **Engagement Type:** Authorized Security Assessment (CTF)

---

## Executive Summary

### Overview
During this penetration test, a hidden endpoint was discovered which revealed the architecture of how both pair of keys and signatures are generated. The RSA seed was not random, it was predictable which makes it easy to exploit because an unauthenticated attacker can exploit it and forge admin messages.  

### Key Findings
- **Critical Vulnerabilities:** 2
  - Predictable RSA Key Generation
  - Passwordless Authentication

### Risk Rating
**Overall Risk Level: CRITICAL** 
They can forge ANY NEW message as admin = impersonate admin to the system and users.

### Recommendation Summary
For Predictable Seed issue:
- Use os.urandom() or secrets.token_bytes() for random seed generation
- Never base keys on predictable data like username

For Passwordless Login issue:
- Implement strong password authentication
- 

---

## 1. Scope & Methodology

### Scope
**In Scope:** 
- Target: 10.48.160.162:5000

### Testing Methodology
**Tools Used:**
- nmap (port scanning & service identification)
- ffuf (directory enumeration)

**Testing Phases:**
1. Reconnaissance - network mapping and endpoint discovery
2. Vulnerability Identification - Passwordless login page and architecture of how keys and signature are generated
3. Exploitation - RCE delivery and command execution
4. Post-Exploitation - flag retrieval and documentation

---

## 2. Reconnaissance & Discovery

### Network Scanning
**Command:** `sudo nmap -sC -sV 10.48.160.162`

**Results:**
- Port 22: (SSH)
- Port 5000: (Flask web server)

### Web Application Enumeration
**Observations:**
- Enumerated the `debug` endpoint
- No password in login page

---

## 3. Vulnerability Details

### Vulnerability 1: Predictable RSA Key Generation
**Type:** Cryptographic Weakness / Weak Key Generation

#### Description
Seeds are based on username + hardcoded string. Anyone who looked up in the website for all the users and generate a seed exact same. This should be replaced by random seed generation which will not only secure the architecture of the cryptography but will also prevent from attackers. 

#### Technical Details
- **Affected Component:** Key generation service/algorithm
- **Attack Vector:** `/debug` reveals seed pattern
- **Authentication Required:** No
- **User Interaction:** No
- **Seed Pattern:** `{username}_lovenote_2026_valentine`
- **Impact:** Attacker can reconstruct private keys for ANY user

#### CVSS v3.1 Score
**Score: [9.8] - CRITICAL**

#### Impact
- **Confidentiality:** They can read any admin messages, private communications, user data that admin can access
- **Integrity:** They can forge messages as admin, fake admin announcements, manipulate message history
- **Availability:** They can send false admin messages disrupting trust, spam as admin, delete messages, lock accounts

#### Proof of Concept (High-Level)
1. Visit the site at `http://10.48.160.162:5000`
2. Visit the `/debug` endpoint to understand the cryptographic architecture
3. Create both keys and a digital signature with private key
4. Verify a message in '/verify' endpoint

---

### Vulnerability 2: Passwordless Authentication
**Type:** Authentication Bypass / Broken Authentication

#### Description
The vulnerability is in login form which only requires username, no password. Why is this a problem? Anyone can login as anyone. It should contain both username and password (strong password mechanism).

#### Technical Details
- **Affected Component:** Login form at `/login`
- **Attack Vector:** HTTP POST to `/login` with only username parameter
- **Authentication Required:** No
- **User Interaction:** Yes
- **Impact:** Unauthenticated attacker can login as ANY user (admin, regular users, etc.)

#### Impact
- **Confidentiality:** - They can access anyone's account
- **Integrity:** - They can modify messages or give misleading message
- **Availability:** - They can send false admin messages disrupting trust, spam as admin, delete messages, lock accounts

---

## 4. Attack Chain / Timeline of Compromise

### Phase 1: Initial Reconnaissance (5 minutes)
- `sudo nmap -sC -sV 10.48.160.162`
- `ffuf -u http://10.48.160.162:5000/FUZZ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt`

### Phase 2: Vulnerability Discovery (5 - 10 minutes)
- Found the `/debug` endpoint
- It revealed seed pattern for key generation

### Phase 3: Exploitation (~1 hour)
- Reconstructed the keys, forged signature, verified on `/verify`
- Flag captured

### Total Compromise Time: ~1.5 hours

---

## 5. Impact Assessment

### Business Impact
Compromise of the application allows attackers to impersonate the system administrator and send fraudulent messages to all users. This undermines user trust in the platform, as attackers can inject misleading information, false announcements, or malicious directives appearing to come from admin. The ability to forge cryptographically verified admin messages could lead to social engineering attacks, data manipulation requests, or instructions that trick users into compromising their own security. This would result in severe reputational damage, loss of user confidence, and potential legal liability under data protection regulations.

### Technical Impact
- **Complete Authentication Bypass:** Attacker gains admin access without credentials via passwordless login
- **Message Integrity Compromise:** All admin messages can be forged with valid cryptographic signatures
- **System Impersonation:** Attacker can send messages appearing authentically from system administrator
- **Key Reconstruction:** Private keys for any user can be derived offline using predictable seed pattern
- **Signature Forgery:** Attacker can create valid digital signatures for any user without their private key

### Affected Assets
- Admin account and privileges
- User message database
- Public message feed (accessible to all users)
- System integrity and trust model
- User authentication and authorization system
- Cryptographic key generation service

---

## 6. Remediation Recommendations

### Priority 1: Critical (Immediate Action Required)

#### 6.1 Implement Secure Random Key Generation
- **Action:** Replace predictable seed-based key generation with cryptographically secure random generation using `os.urandom()` or `secrets.token_bytes()`
- **Timeline:** Within 24-48 hours
- **Details:**
  - Remove username-based seed pattern entirely
  - Use high-entropy random values for RSA prime generation
  - Regenerate all existing user keys immediately
  - Store keys securely in encrypted database

#### 6.2 Implement Strong Password Authentication
- **Action:** Add mandatory password authentication to login form
- **Timeline:** Immediate (same deployment)
- **Implementation:**
  - Require strong passwords (minimum 12 characters, complexity requirements)
  - Use bcrypt or Argon2 for password hashing
  - Implement rate limiting on login attempts (5 attempts per 15 minutes)
  - Add account lockout after failed attempts

---

### Priority 2: High (Short-term)

#### 6.2.1 Add Message Signature Verification Logging
- **Action:** Log all signature verification attempts and failures
- **Timeline:** 1 week
- **Details:** Monitor for forged message attempts, alert on suspicious patterns

#### 6.2.2 Implement Access Controls
- **Action:** Restrict `/debug` endpoint to authenticated admin users only
- **Timeline:** 1 week
- **Details:** Remove or restrict debug information exposure in production

#### 6.2.3 Key Rotation Policy
- **Action:** Implement periodic key rotation and revocation mechanism
- **Timeline:** 2 weeks
- **Details:** Allow users to regenerate keys, maintain key history for audit trails

---

## 7. Lessons Learned & Observations

### What Worked Well
- Systematic reconnaissance using nmap and ffuf identified all endpoints quickly
- Manual testing of login form revealed authentication weakness immediately
- Debug endpoint discovery exposed architectural details critical to exploitation
- Python scripting with cryptography library efficiently reconstructed keys and forged signatures
- `/verify` endpoint provided clear feedback on signature validity

### Areas for Improvement
- Debug endpoint should never be accessible in production; requires strict access controls
- Key generation algorithm relied entirely on predictable data; should use cryptographic randomness
- Authentication design skipped password requirement entirely; fundamental security control missing
- No signature audit logging makes detection of forged messages difficult

### Tools Effectiveness
- **nmap:** Essential for service identification and port discovery
- **ffuf:** Effective for systematic endpoint enumeration
- **cryptography library:** Powerful for RSA operations and key reconstruction
- **Manual testing:** Most valuable for identifying logic flaws in authentication and message verification

---

## 8. Conclusion

The LoveNote application contains critical vulnerabilities in both cryptographic key generation and authentication that allow unauthenticated remote exploitation. The predictable RSA seed pattern, combined with passwordless login, enables attackers to completely compromise system integrity by forging admin messages with valid cryptographic signatures. Immediate implementation of secure random key generation and strong password authentication is essential to prevent message forgery attacks and unauthorized admin access.

**Risk Level:** CRITICAL - Immediate action required

---

## Appendix A: Command Reference

### Reconnaissance
```bash
sudo nmap -sC -sV 10.48.160.162
ffuf -u http://10.48.160.162:5000/FUZZ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```

### Exploitation
```bash
# Python script for key reconstruction and signature forgery
python3 signedmessages_exploit.py

# Input: admin
# Output: Reconstructed private key and forged signature

# Manual verification on /verify endpoint
# Sender: admin
# Message: I am the admin
# Signature: [hex output from script]
```

---

**Report Prepared By:** Bishal Kumar Baishya
**Date:** October 1, 2026
**Classification:** Internal Use