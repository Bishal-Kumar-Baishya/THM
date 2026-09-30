# SpeedChat - CTF Penetration Testing

A comprehensive penetration testing report and walkthrough of the **SpeedChat** CTF challenge on TryHackMe, demonstrating exploitation of unrestricted file upload vulnerability leading to remote code execution and complete system compromise.

---

## 📋 Overview

**Target:** Flask web application (Chat messaging platform)  
**IP Address:** 10.49.156.222:5000  
**Vulnerabilities Discovered:** 1 Critical  
**Outcome:** Complete system compromise (root access)  
**Time to Compromise:** ~30 minutes  

---

## 🎯 Vulnerabilities Identified

| Vulnerability | Severity | CVSS Score |
|---------------|----------|-----------|
| Unrestricted File Upload with Code Execution | CRITICAL | 9.8 |

---

## 📁 Repository Structure
```
speedchat/
├── README.md # This file - Quick reference
├── REPORT.md # Professional penetration testing report
├── WALKTHROUGH.md # Step-by-step exploitation guide
```


---

## 🔍 Quick Summary

### Attack Chain
1. **Reconnaissance** → Network scanning (nmap, ffuf)
2. **Vulnerability Discovery** → Unrestricted file upload identified
3. **Initial Access** → Python reverse shell upload
4. **Reverse Shell** → Interactive bash shell via netcat
5. **Flag Retrieval** → Root access achieved, flag captured

### Key Findings
- ✅ Unauthenticated file upload with no restrictions
- ✅ Flask automatically executes uploaded Python files
- ✅ RCE achieved as root user
- ✅ Complete system compromise in ~30 minutes
- ✅ No authentication or user interaction required

---

## 🚀 Getting Started

### Prerequisites
- Kali Linux / Penetration Testing Distro
- Tools installed:
  - `nmap` - Network scanning
  - `ffuf` - Directory enumeration
  - `netcat` - Reverse shell access
  - `python3` - Payload creation

### Usage
Read the documentation in order:

1. **[REPORT.md](./REPORT.md)** - Professional findings and analysis
2. **[WALKTHROUGH.md](./WALKTHROUGH.md)** - Step-by-step exploitation guide

---

## 📝 Exploitation Steps

### Step 1: Reconnaissance
```bash
sudo nmap -sC -sV 10.49.156.222
ffuf -u http://10.49.156.222:5000/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

### Step 2: Create Python Reverse Shell
Create `shell.py` with interactive bash shell payload

### Step 3: Setup Listener
```bash
nc -lvnp 9001
```

### Step 4: Upload and Execute
Upload `shell.py` via the `/upload_profile_pic` endpoint

### Step 5: Flag Retrieval
```bash
cat flag.txt
```

---

## 🤝 Disclaimer

This penetration testing exercise was conducted as part of an authorized CTF challenge on TryHackMe. The techniques and tools described are for **educational and authorized testing purposes only**. Unauthorized access to computer systems is illegal.

---

## ✍️ Author

**Bishal Kumar Baishya**  
Cybersecurity Student | Penetration Testing Focus  
Portfolio: [GitHub Link](https://github.com/Bishal-Kumar-Baishya)  
LinkedIn: [LinkedIn Profile](https://www.linkedin.com/in/bishal-kumar-baishya-022b56412/)

---

## 📄 License

This project is for educational purposes. All documentation and findings are available for security professionals and students.

---

## 🔗 Quick Links

- 📖 [Full Penetration Testing Report](./REPORT.md)
- 🎯 [Step-by-Step Walkthrough](./WALKTHROUGH.md)
- 💼 [Portfolio](https://github.com/Bishal-Kumar-Baishya)

---

**Last Updated:** September 30, 2026  
**Status:** Complete ✅