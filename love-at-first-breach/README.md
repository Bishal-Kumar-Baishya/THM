# Love at First Breach CTF Series

A comprehensive collection of penetration testing challenges from the TryHackMe "Love at First Breach" series. Each challenge demonstrates different vulnerability types, exploitation techniques, and security assessment methodologies.

---

## 📋 Series Overview

**Platform:** TryHackMe  
**Series:** Love at First Breach  
**Focus Areas:** Web Security, Data Exposure, RCE, Privilege Escalation, Authentication  

---

## 🎯 Challenges

### 1. ValenFind

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

**GitHub:** [View Full Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/valenfind/REPORT.md)

---

### 2. DeepIntoMyHeart

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

**GitHub:** [View Full Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/DeepIntoMyHeart/REPORT.md)

---

### 3. CupidBot

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

**GitHub:** [View Full Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/CupidBot/WALKTHROUGH.md)

---

### 4. TryHeartMe

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

**GitHub:** [View Full Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/TryHeartMe/REPORT.md)

---

### 5. SpeedChat

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
**GitHub:** [View Full Report](https://github.com/Bishal-Kumar-Baishya/THM/tree/main/love-at-first-breach/SpeedChat/PENTEST_REPORT.md)

---

### 6. Romance and Co.

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
**GitHub:** [View Full Report](https://github.com/Bishal-Kumar-Baishya/THM/tree/main/love-at-first-breach/Romance%26co/REPORT.md)

---

## 📊 Challenge Comparison

| Challenge | Difficulty | Vulnerability Type | Category | Time | Status |
|-----------|------------|-------------------|----------|------|--------|
| ValenFind | Medium | Local File Inclusion (LFI) | Web | ~90 min | ✅ |
| DeepIntoMyHeart | Easy | Data Exposure | Reconnaissance | ~45 min | ✅ |
| CupidBot | Easy | Prompt Injection | AI/Web | ~10 min | ✅ |
| TryHeartMe | Easy | JWT Manipulation | Authentication | ~15 min | ✅ |
| SpeedChat | Easy | File Upload RCE | Web | ~30 min | ✅ |
| Romance and Co. | Medium | RCE + Privilege Escalation | Web/System | ~150 min | ✅ |

---

## 🛠️ Tools & Techniques Used Across Series

**Reconnaissance:**
- nmap (network scanning)
- gobuster/ffuf (directory enumeration)
- nuclei (vulnerability scanning)
- curl/wget (manual probing)
- Browser DevTools (token extraction)

**Exploitation:**
- Python scripting (custom exploits)
- react2shell (RCE delivery)
- netcat (reverse shells)
- jwt.io (JWT decoding/manipulation)
- Manual vulnerability testing
- Prompt injection (social engineering)
- File upload exploitation

**Privilege Escalation:**
- sudo enumeration (`sudo -l`)
- SUID binary analysis
- Kernel exploit research
- Service misconfiguration abuse
- Token claim manipulation

**Reverse Shell Development:**
- Socket programming (Python)
- File descriptor redirection (os.dup2)
- Interactive bash shells
- Netcat listeners

---

## 📁 Repository Structure
```
love-at-first-breach/
├── README.md (this file - series overview)
├── ValenFind/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── DeepIntoMyHeart/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── CupidBot/
│ ├── README.md
│ └── WALKTHROUGH.md
├── TryHeartMe/
│ ├── README.md
│ ├── REPORT.md
│ └── WALKTHROUGH.md
├── SpeedChat/
│ ├── README.md
│ ├── PENTEST_REPORT.md
│ └── WALKTHROUGH.md
└── Romance&co/
  ├── README.md
  ├── REPORT.md
  └── WALKTHROUGH.md
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

### Authentication & Authorization
✅ JWT token structure and implementation  
✅ Signature verification requirements  
✅ Role-based access control (RBAC)  
✅ Token claim validation  
✅ Backend authentication checks  

### Reconnaissance & Enumeration
✅ Network scanning (nmap)  
✅ Directory brute-forcing (gobuster, ffuf)  
✅ Automated vulnerability detection (nuclei)  
✅ Manual web application testing  
✅ Source code analysis  
✅ Storage inspection and token extraction  

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

### Documentation & Reporting
✅ Professional penetration testing reports  
✅ CVSS score assessment  
✅ Detailed technical walkthroughs  
✅ Remediation recommendations  
✅ Lessons learned analysis  

---

## 📈 Learning Progression

**Challenge 1 (Easy):** Data Exposure  
→ Understand reconnaissance and information gathering

**Challenge 2 (Easy):** Prompt Injection  
→ Learn AI system vulnerabilities and social engineering

**Challenge 3 (Easy):** JWT Manipulation  
→ Learn authentication vulnerabilities and token exploitation

**Challenge 4 (Easy):** File Upload RCE  
→ Learn unrestricted upload exploitation and reverse shell development

**Challenge 5 (Medium):** LFI Exploitation  
→ Learn vulnerability exploitation and file system access

**Challenge 6 (Medium):** RCE + Privilege Escalation  
→ Master complete system compromise and advanced techniques

---

## 🔑 Key Takeaways Across Series

1. **Reconnaissance is Critical** - Detailed recon saves exploitation time
2. **Automate Where Possible** - Tools like nuclei identify vulns faster than manual testing
3. **Defense in Depth Fails** - Combination of multiple weaknesses = total compromise
4. **Privilege Escalation Vectors** - sudo, SUID, kernel vulns, and token manipulation are common paths to root
5. **Documentation Matters** - Professional reports are as important as technical skills
6. **Social Engineering Works** - Even AI systems can be fooled by authority claims
7. **Never Trust User Input** - Especially for authentication and authorization claims
8. **Match Payload Language to Backend** - Python shells for Python apps, bash for Linux, etc.
9. **Interactive Shells are Better** - More reliable than command-based shells for persistence
10. **Always Validate File Uploads** - Use magic bytes + library verification, never just extensions
11. **AI Security = Application Security** - Same principles apply to LLM-based systems
12. **Always Verify Cryptographic Signatures** - JWTs must validate with server secret key

---

## 🤝 Disclaimer

All challenges were completed as part of authorized CTF assessments on TryHackMe. The techniques and tools described are for **educational and authorized testing purposes only**. Unauthorized access to computer systems is illegal.

---

## ✍️ Author

**Bishal Kumar Baishya**  
Cybersecurity Student | Penetration Testing Focus  

**Contact:**
- Portfolio: [GitHub](https://github.com/Bishal-Kumar-Baishya)
- LinkedIn: [Profile](https://www.linkedin.com/in/bishal-kumar-baishya-022b56412/)

---

## 📄 License

Educational purposes only. All documentation and findings are available for security professionals and students.

---

## 🔗 Quick Navigation

- 📖 [ValenFind Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/valenfind/REPORT.md)
- 📖 [DeepIntoMyHeart Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/DeepIntoMyHeart/REPORT.md)
- 📖 [CupidBot Walkthrough](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/CupidBot/WALKTHROUGH.md)
- 📖 [TryHeartMe Report](https://github.com/Bishal-Kumar-Baishya/THM/blob/main/love-at-first-breach/TryHeartMe/REPORT.md)
- 📖 [SpeedChat Report](https://github.com/Bishal-Kumar-Baishya/THM/tree/main/love-at-first-breach/SpeedChat/REPORT.md)
- 📖 [Romance and Co. Report](https://github.com/Bishal-Kumar-Baishya/THM/tree/main/love-at-first-breach/Romance%26co/REPORT.md)
- 🐱 [GitHub Repository](https://github.com/Bishal-Kumar-Baishya/love-at-first-breach)

---

**Last Updated:** September 30, 2026