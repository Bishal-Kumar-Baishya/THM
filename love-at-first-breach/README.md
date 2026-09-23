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

