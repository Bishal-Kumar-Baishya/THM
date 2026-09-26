# TryHeartMe - JWT Privilege Escalation CTF

A simple but effective demonstration of JWT token manipulation to achieve unauthorized privilege escalation and admin account access.

---

## 📋 Overview

**Platform:** TryHackMe  
**Series:** Love at First Breach  
**Category:** Web / Authentication  
**Difficulty:** Easy  
**Vulnerability:** JWT Token Manipulation  
**Status:** ✅ Complete  

---

## 🎯 Challenge Summary

TryHeartMe is a Flask web application with JWT-based authentication. **The vulnerability**: JWT tokens don't validate signatures properly, allowing attackers to modify claims (user role, credit balance) and gain unauthorized access.

---

## 🔓 Vulnerability

| Aspect | Detail |
|--------|--------|
| **Type** | JWT Manipulation / Privilege Escalation |
| **Root Cause** | No signature verification on token claims |
| **Attack Vector** | Decode JWT payload, modify claims, re-encode |
| **Impact** | Unauthorized admin access |
| **Difficulty to Exploit** | Trivial - no cryptography required |

---

## 💡 The Exploit

**Three Simple Steps:**
1. Extract JWT token from browser storage
2. Decode payload and modify claims (user→admin, credit→9999999)
3. Replace token and gain admin access

---

## 🛡️ Why This Is Critical

- No signature verification on tokens
- User claims accepted without validation
- Role-based access control bypassed via token manipulation
- Privilege escalation in seconds

---

## 🎓 Key Learning

**JWT Security Principles:**
- JWTs must validate signatures with server secret key
- Token claims must never be trusted without verification
- Alg=none should never be accepted in production
- Role-based access control requires backend validation

---

## 📚 Real-World Impact

This vulnerability mirrors real issues in:
- E-commerce platforms (modifying user role/credit)
- SaaS applications (privilege escalation)
- API authentication systems
- Multi-tenant applications

---

## 🤝 Disclaimer

This challenge is part of an authorized CTF on TryHackMe. JWT manipulation techniques should only be used in authorized testing environments.

---

## ✍️ Author

**Bishal Kumar Baishya**  
Cybersecurity Student | Penetration Testing Focus  
Portfolio: [GitHub](https://github.com/Bishal-Kumar-Baishya)  

---

**Last Updated:** September 26, 2026  
**Status:** Complete ✅