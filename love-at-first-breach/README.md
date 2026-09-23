# Love at First Breach CTF Series

A comprehensive collection of penetration testing challenges from the TryHackMe "Love at First Breach" series. Each challenge demonstrates different vulnerability types, exploitation techniques, and security assessment methodologies.

---

## 📋 Series Overview

**Platform:** TryHackMe  
**Series:** Love at First Breach 
**Focus Areas:** Web Security, Data Exposure, RCE, Privilege Escalation  

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

**GitHub:** [View Full Report](./ValenFind/)

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

**GitHub:** [View Full Report](./DeepIntoMyHeart/)

---

### 3. Romance and Co.

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
**GitHub:** [View Full Report](./Romance&co/)

### 4. CupidBot

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

**Vulnerability Details:**
- Bot accepts user claims of admin status without verification
- No identity checking before revealing sensitive data
- Social engineering is effective against poorly designed systems

**GitHub:** [View Full Report](./CupidBot/)

---

## 📊 Challenge Comparison

| Challenge | Difficulty | Vulnerability Type | Category | Time | Status |
|-----------|------------|-------------------|----------|------|--------|
| ValenFind | Medium | Local File Inclusion (LFI) | Web | ~90 min | ✅ |
| DeepIntoMyHeart | Easy | Data Exposure | Reconnaissance | ~45 min | ✅ |
| CupidBot | Easy | Prompt Injection | AI/Web | ~10 min | ✅ |
| Romance and Co. | Medium | RCE + Privilege Escalation | Web/System | ~150 min | ✅ |

---

## 🛠️ Tools & Techniques Used Across Series

**Reconnaissance:**
- nmap (network scanning)
- gobuster (directory enumeration)
- nuclei (vulnerability scanning)
- curl/wget (manual probing)

**Exploitation:**
- Python scripting (custom exploits)
- react2shell (RCE delivery)
- netcat (reverse shells)
- Manual vulnerability testing
- Prompt injection (social engineering)

**Privilege Escalation:**
- sudo enumeration (`sudo -l`)
- SUID binary analysis
- Kernel exploit research
- Service misconfiguration abuse

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

### Reconnaissance & Enumeration
✅ Network scanning (nmap)  
✅ Directory brute-forcing (gobuster)  
✅ Automated vulnerability detection (nuclei)  
✅ Manual web application testing  
✅ Source code analysis  

### Exploitation & Post-Exploitation
✅ Remote Code Execution (RCE) delivery  
✅ Reverse shell establishment  
✅ Privilege escalation techniques  
✅ Post-exploitation persistence  
✅ Flag retrieval and documentation  
✅ Social engineering against AI systems  

### Documentation & Reporting
✅ Professional penetration testing reports  
✅ CVSS score assessment  
✅ Detailed technical walkthroughs  
✅ Remediation recommendations  
✅ Lessons learned analysis  

---

## 🔑 Key Takeaways Across Series

1. **Reconnaissance is Critical** - Detailed recon saves exploitation time
2. **Automate Where Possible** - Tools like nuclei identify vulns faster than manual testing
3. **Defense in Depth Fails** - Combination of multiple weaknesses = total compromise
4. **Privilege Escalation Vectors** - sudo, SUID, kernel vulns are common paths to root
5. **Documentation Matters** - Professional reports are as important as technical skills
6. **Social Engineering Works** - Even AI systems can be fooled by authority claims
7. **AI Security = Application Security** - Same principles apply to LLM-based systems

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

- 📖 [ValenFind Report](./ValenFind/REPORT.md)
- 📖 [DeepIntoMyHeart Report](./DeepIntoMyHeart/REPORT.md)
- 📖 [CupidBot Walkthrough](./CupidBot/WALKTHROUGH.md)
- 📖 [Romance and Co. Report](./Romance&co/REPORT.md)
- 🐱 [GitHub Repository](https://github.com/Bishal-Kumar-Baishya/love-at-first-breach)

---

**Last Updated:** September 23, 2026  
