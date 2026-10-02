# Signed Messages - CTF Walkthrough

**Target:** 10.48.160.162  
**Difficulty:** Medium  
**Category:** Web, Crypto  
**Time to Compromise:** ~1.5 hours

---

## Executive Summary

**Attack Chain:**
1. Discover passwordless authentication (no password field in login)
2. Login as `admin` without credentials
3. Access `/debug` endpoint (unauthenticated) → reveals RSA key generation algorithm
4. Identify predictable seed pattern: `{username}_lovenote_2026_valentine`
5. Reconstruct admin's private RSA key offline
6. Forge a message signed by admin
7. Submit forged signature to `/verify` → Flag captured

---

## Prerequisites

### Tools Required
- `nmap` — Network port scanning
- `ffuf` — Web endpoint enumeration
- Python 3 with libraries: `cryptography`, `sympy`, `hashlib`
- Browser (Firefox/Chrome for manual verification)

### Concepts Required
- RSA cryptography (public/private key pairs, digital signatures)
- SHA256 hash functions
- How digital signatures work (sign with private key, verify with public key)
- Modular arithmetic basics

---

## Step 1: Reconnaissance

### 1.1 Network Scanning

Identify open ports and running services:

```bash
sudo nmap -sC -sV 10.48.160.162
```

**Results:**
- 22/tcp open ssh OpenSSH 7.6p1
- 5000/tcp open flask Flask web application


**Finding:** Web application running on port 5000 (primary target)

### 1.2 Web Application Enumeration

Discover available endpoints:

```bash
ffuf -u http://10.48.160.162:5000/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

**Key Endpoints Discovered:**
```
about (200 OK)
compose (200 OK)
dashboard (200 OK)
debug (200 OK) ← CRITICAL: No authentication required
login (200 OK)
logout (200 OK)
messages (200 OK)
register (200 OK)
verify (200 OK)
```


**Initial Observation:** The `/debug` endpoint exists and is publicly accessible — potential information leak.

---

## Step 2: Authentication Analysis

### 2.1 Login Form Investigation

Navigated to `http://10.48.160.162:5000/login` in the browser and examined the HTML form.

**Critical Discovery:** The login form contains **only a username field** — no password field exists.

```html
<!-- Observed HTML structure -->
<form method="POST">
    <input type="text" name="username" placeholder="Username" required>
    <button type="submit">Login</button>
</form>
```

**Implication:** Authentication is not password-based; vulnerability exists.

### 2.2 User Enumeration

Registered a test account with username `sam_sepiol` to understand the app flow.

Attempted login with username `admin` (common default admin username) → **Login succeeded without any password**.

**Vulnerability Confirmed:** Passwordless authentication allows anyone to impersonate any user, including `admin`.

---

## Step 3: Debug Endpoint Exploitation

### 3.1 Accessing the `/debug` Endpoint

Navigated to `http://10.48.160.162:5000/debug` (no authentication required).

**Output Revealed:**
- Key generation algorithm in plain text
- Seed pattern for RSA key generation
- Process for calculating primes p and q
- Values of public exponent (e = 65537) and key size

### 3.2 Understanding Key Generation Architecture

The debug output showed:
```
Seed Pattern: {username}_lovenote_2026_valentine
```

**Key Generation Process:**
- seed_string = "{username}_lovenote_2026_valentine"
- p = nextprime(SHA256(seed_string))
- q = nextprime(SHA256(seed_string + b"pki"))
- n = p × q
- e = 65537
- d = modular_inverse(e, φ(n)) where φ(n) = (p-1)(q-1)
- Private key = (n, d, p, q, ...)


**Critical Weakness:** The seed is **completely predictable** because it's based only on the username, which is public information.

**Implication:** An attacker can reconstruct ANY user's private key offline, without accessing the server's key storage.

---

## Step 4: Key Reconstruction

### 4.1 Why This Attack Works

**RSA Math Overview:**
- **Public Key:** (n, e) — used to verify signatures
- **Private Key:** (n, d) — used to create signatures
- **Security Assumption:** It's computationally hard to factor n back into p and q

**However:** If we can predict p and q (via the predictable seed), we can:
1. Calculate n = p × q ourselves
2. Calculate φ(n) = (p-1)(q-1)
3. Calculate d = e⁻¹ mod φ(n) (the private exponent)
4. Reconstruct the entire private key

**Result:** We can sign messages as if we were the admin.

### 4.2 Python Exploit Script

Create a file `exploit.py`:

```python
#!/bin/python3

import hashlib
from sympy import nextprime
from cryptography.hazmat.primitives.asymmetric import rsa, padding, utils
from cryptography.hazmat.primitives import hashes

username = input("[*] Enter the username: ")

seed_str = f"{username}_lovenote_2026_valentine"
seed_bytes = seed_str.encode()
hash1 = hashlib.sha256(seed_bytes).digest()
p_candidate = int.from_bytes(hash1, byteorder='big')
p = nextprime(p_candidate)

new_seedBytes = seed_bytes + b"pki"
hash2 = hashlib.sha256(new_seedBytes).digest()
q_candidate = int.from_bytes(hash2, byteorder='big')
q = nextprime(q_candidate)

n = p * q
e = 65537

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)

dmp1 = d % (p - 1)
dmq1 = d % (q - 1)
iqmp = pow(q, -1, p)

public_numbers = rsa.RSAPublicNumbers(e=e, n=n)
private_numbers = rsa.RSAPrivateNumbers(
    p=p, q=q, d=d, dmp1=dmp1, dmq1=dmq1, iqmp=iqmp, public_numbers=public_numbers,
)

private_key = private_numbers.private_key()
print(f"[+] Successfully constructed key object: {private_key}")

message = "I am the admin"
message_bytes = message.encode()
hash3 = hashlib.sha256(message_bytes).digest()

signature = private_key.sign(
    data=hash3,
    padding=padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH
    ),
    algorithm=utils.Prehashed(hashes.SHA256())
)

print(f"[+] Signature created successfully: {signature.hex()}")
```

### 4.3 Running the Exploit

```bash
python3 exploit.py
```

**Input:** `admin`

**Output:** [REDACTED]


**What Happened:**
- Script reconstructed p and q using the predictable seed
- Calculated private key (d) from p, q, and e
- Hashed message `"I am the admin"` with SHA256
- Signed the hash using RSA-PSS padding (probabilistic, provides randomness)
- Generated hex signature output

---

## Step 5: Message Forgery & Verification

### 5.1 Accessing the `/verify` Endpoint

Navigated to `http://10.48.160.162:5000/verify` in the browser.

### 5.2 Submitting the Forged Signature

Filled out the verification form with:

| Field | Value |
|-------|-------|
| **Sender** | `admin` |
| **Message** | `I am the admin` |
| **Signature** | `[paste hex signature from exploit output]` |

Clicked **Submit**.

### 5.3 Server Response

The server verified the signature against the admin's **public key** (which it already has).

**Result:** ✅ Signature verified as VALID

**Server Message:** "You successfully forged an admin signature!"

**Flag Displayed:** `THM{PR3D1CT4BL3_S33D5_BR34K_H34RT5}`

---

## Key Learnings & Security Implications

### 1. Never Use Predictable Data in Cryptographic Seeds

**The Problem:**
- Seed = username + hardcoded string is **deterministic**
- Attacker can compute it without access to the system
- Private key generation becomes offline-brute-forceable

**The Fix:**
- Use cryptographically secure random number generators (e.g., `os.urandom()`)
- Never derive keys from usernames, timestamps, or other public data
- If keys must be generated on login, use server-side secure randomness

### 2. Passwordless Authentication is a Critical Flaw

**The Problem:**
- Login form without password field = anyone can log in as anyone
- Even if signatures work perfectly, authentication is broken
- Violates AAA principle (Authentication, Authorization, Accounting)

**The Fix:**
- Implement multi-factor authentication (password + 2FA)
- Never trust username alone for identity verification
- Even "secure" features (like digital signatures) fail if authentication doesn't exist

### 3. Debug Endpoints Must Never Be Public

**The Problem:**
- `/debug` revealed the entire key generation algorithm
- Attackers don't need to reverse-engineer; developers hand them the blueprint
- "Security through obscurity" doesn't work, but transparency has limits

**The Fix:**
- Remove debug endpoints in production
- If debugging is needed, require authentication + authorization
- Never log sensitive architectural details (key generation, algorithm choices)
- Use environment-based configuration to disable debugging

### 4. Digital Signatures Are Only as Secure as Their Keys

**The Problem:**
- RSA signature verification is mathematically sound
- But if the private key is compromised or predictable, signatures are worthless
- The `/verify` endpoint correctly verified our forged signature (signature crypto was fine; the key was the issue)

**The Fix:**
- Protect private key generation with strong randomness
- Consider using Hardware Security Modules (HSM) for key storage
- Rotate keys regularly
- Never assume signature verification = authentication

---


---

## Timeline

| Phase | Duration | Activity |
|-------|----------|----------|
| Reconnaissance | 10 min | nmap + ffuf enumeration |
| Analysis | 15 min | Login form inspection + /debug access |
| Exploitation | 30 min | Python script development + testing |
| Verification | 5 min | Form submission + flag capture |
| **Total** | **~1 hour** | — |

---

## Conclusion

This challenge demonstrates how multiple seemingly-small security failures compound into complete system compromise:

- ❌ Weak authentication (no password)
- ❌ Information disclosure (debug endpoint)
- ❌ Predictable cryptography (seed pattern)

**Result:** Full system compromise despite using "secure" digital signatures.

The lesson: **Security is a chain; it's only as strong as its weakest link.**