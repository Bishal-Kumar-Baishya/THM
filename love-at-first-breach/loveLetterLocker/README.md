# Love Letter Locker - CTF Challenge Write-Up

## Overview

**Love Letter Locker** is an easy-difficulty TryHackMe web security challenge that demonstrates the critical vulnerability of **IDOR (Insecure Direct Object Reference)** combined with **sequential/predictable resource identifiers**.

**Challenge Link:** [TryHackMe - Love Letter Locker](https://tryhackme.com/)  
**Difficulty:** Easy  
**Category:** Web Security, Authorization  
**Time to Solve:** ~15 minutes

---

## What You'll Learn

This challenge teaches fundamental web security concepts through practical exploitation:

### Core Vulnerabilities

1. **IDOR (Insecure Direct Object Reference)** — Accessing resources without permission verification
2. **Broken Access Control** — Missing authorization checks on sensitive endpoints
3. **Sequential ID Predictability** — Using enumerable IDs enables easy exploitation
4. **Missing Ownership Verification** — Not checking if the requester owns the resource

### Security Concepts

- **Authorization vs Authentication** — Who you are vs. what you can access
- **Direct Object References** — Database IDs in URLs
- **Predictable vs Random IDs** — Impact on security
- **Privilege Escalation via IDOR** — Accessing admin/other user resources
- **OWASP A01:2021 — Broken Access Control**

---

## Challenge Description

Love Letter Locker is a romantic web application where users can write, store, and read their Valentine's love letters. The app promises privacy: "For your eyes only."

**The Reality:** Due to lack of authorization checks, ANY user can read ANY other user's private letters simply by changing a number in the URL.

---

## Quick Start

### Prerequisites

**System Requirements:**
- Linux/macOS or WSL on Windows
- Python 3.8+
- Access to TryHackMe

**Tools Used:**
```bash
nmap           # Network scanning
ffuf           # Web enumeration
curl/wget      # HTTP requests
Browser        # Web interaction
```

### Exploitation Steps

1. **Start the TryHackMe machine** and note the target IP

2. **Scan the target:**
```bash
   nmap -sC -sV TARGET_IP
```

3. **Enumerate web endpoints:**
```bash
   ffuf -u http://TARGET_IP:5000/FUZZ -w /path/to/wordlist
```

4. **Register a test account**

5. **Create a love letter** and note the URL format (e.g., `/letter/3`)

6. **Test IDOR vulnerability:**
```bash
   # Try accessing other users' letters
   curl http://TARGET_IP:5000/letter/1
   curl http://TARGET_IP:5000/letter/2
```

7. **Enumerate all letters with ffuf:**
```bash
   ffuf -u http://TARGET_IP:5000/letter/FUZZ -w <(seq 1 50)
```

8. **Find and retrieve the flag** from the accessible letters

---

## Security Lessons

### Lesson 1: Authorization is Not Optional

**The Principle:**
Every resource access requires TWO checks:
1. **Authentication:** "Who are you?" (Username/password)
2. **Authorization:** "Are you allowed to access this?" (Ownership check)

**Apply This:**
- Never assume authenticated users can access all resources
- Check permissions on EVERY protected endpoint
- Default to DENY, not ALLOW

---

### Lesson 2: Predictable IDs are Dangerous

**The Principle:**
If someone can guess the resource ID, they can access it (if no auth check).

**Why Predictable:**
- Sequential numbers (1, 2, 3, 4...)
- Timestamps (2026-10-07-001, 2026-10-07-002...)
- Email-based (user@email.com → easy to guess related accounts)
- Logical patterns (user1, user2, admin1, admin2...)

**Real-World Examples:**

| ID Type | Example | Risk |
|---------|---------|------|
| Sequential | `/order/1000`, `/order/1001` | 🔴 CRITICAL |
| Timestamp | `/invoice/20261007001` | 🔴 CRITICAL |
| Email | `/profile/admin@company.com` | 🟠 HIGH |
| UUID | `/document/a7f3-k9c2...` | 🟢 SAFE |

---

### Lesson 3: Combine Multiple Security Controls

**Defense in Depth:**
```
Predictable ID + No Auth Check = COMPROMISED
Predictable ID + Auth Check = SECURE
Random ID + No Auth Check = SEMI-SECURE
Random ID + Auth Check = MOST SECURE
```


**Apply This:**
- Never rely on a single security control
- Use unpredictable IDs AND authorization checks
- Add rate limiting to prevent bulk enumeration
- Log and monitor resource access
- Implement audit trails

---

## OWASP Top 10 Coverage

This challenge demonstrates:

- **A01:2021 – Broken Access Control**
  - Failure to enforce user permissions
  - Missing authorization on protected resources
  - Vertical privilege escalation (accessing other users' data)

---

## Real-World IDOR Examples

Organizations have been vulnerable to IDOR leading to major breaches:

| Company | Vulnerable Endpoint | Impact | Fixed |
|---------|-------------------|--------|-------|
| Facebook | `/user/{id}/photos` | Access private photos | Yes (2018) |
| Uber | `/trip/{trip_id}` | View passenger routes | Yes (2015) |
| Airbnb | `/booking/{id}` | Read reservation details | Yes (2014) |
| LinkedIn | `/profile/{member_id}` | Scrape profiles | Yes (2012) |
| Twitter | `/user/{id}/followers` | Enumerate private accounts | Yes (ongoing) |

All of these would have been IDOR without proper authorization checks.

---

## Recommended Reading

### IDOR & Access Control

- [OWASP IDOR (Broken Access Control)](https://owasp.org/www-community/attacks/Insecure_Direct_Object_References)
- [PortSwigger - Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Top 10 - A01:2021](https://owasp.org/Top10/)

### Secure ID Generation

- [OWASP - Secure Identifiers](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Coding_Cheat_Sheet.html)
- [Python UUID Documentation](https://docs.python.org/3/library/uuid.html)
- [Random ID Best Practices](https://tools.ietf.org/html/rfc4122)

### Lab Environments

- [HackTheBox Academy - IDOR](https://academy.hackthebox.com/)
- [TryHackMe - Web Security](https://tryhackme.com/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

---

## Common Issues & Troubleshooting

### Issue: "I can't access other users' letters"

**Possible Causes:**
1. Letters use random UUIDs instead of sequential IDs
2. Authorization check is properly implemented
3. App requires different authentication

**Solution:**
- Check if `/letter/1` works at all
- Verify sequential ID pattern exists
- Try accessing while NOT logged in (some apps don't require auth)

---

### Issue: "Getting 404 on /letter/1"

**Possible Causes:**
1. No letters exist with ID 1 yet
2. IDs don't start at 1 (might start at 100, 1000, etc.)
3. The endpoint doesn't exist

**Solution:**
- Create a letter and note the exact ID
- Try IDs near yours: if your letter is `/letter/50`, try `/letter/49`, `/letter/51`
- Use ffuf to find all valid IDs: `ffuf -u http://TARGET/letter/FUZZ -w <(seq 1 100)`

---

### Issue: "I found letter ID 1 but it's not readable"

**Possible Causes:**
1. Letter contains no text (placeholder)
2. Flag is encoded or hidden
3. Need to check response carefully

**Solution:**
- View page source (Ctrl+U) — flag might be in HTML comments
- Check all letters systematically for readable content
- Some letters might be empty; check others

---

## Disclaimer

**Educational Purpose Only:**

This challenge is created for authorized learning on TryHackMe. The techniques described are for **educational and authorized testing purposes only**.

**Legal Boundaries:**
- ❌ DO NOT use IDOR techniques on systems you don't own
- ❌ DO NOT access data without explicit permission
- ❌ DO NOT cause harm or disruption

**DO:**
- ✅ Use knowledge on dedicated lab environments (HTB, TryHackMe)
- ✅ Report IDOR vulnerabilities through responsible disclosure
- ✅ Only test systems with explicit written authorization

**Unauthorized access is illegal** under laws including:
- Computer Fraud and Abuse Act (CFAA) — USA
- Computer Misuse Act — UK
- Similar laws in all countries

---

## Author

**Bishal Kumar Baishya** (Cybersecurity Student)

**Specializations:**
- Offensive Security & Penetration Testing
- Web Application Security
- Access Control & Authorization Testing
- Bug Bounty Hunting

**GitHub:** [GitHub](https://github.com/Bishal-Kumar-Baishya)
**LinkedIn:** [LinkedIn](https://www.linkedin.com/in/bishal-kumar-baishya-022b56412)
---

## License

This write-up is provided for educational purposes. Feel free to reference, share, and learn from it.

**Attribution:** Please credit "Bishal" when referencing this work.

---

**Last Updated:** October 7, 2026
