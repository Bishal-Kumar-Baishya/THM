# TryHeartMe - Security Assessment Report

**Target:** Flask Web Application  
**IP:** 10.49.133.54:5000  
**Objective:** Identify and exploit authentication vulnerabilities  

---

## Executive Summary

The Flask application uses JWT-based authentication without proper signature verification. This allows attackers to decode JWT payloads, modify claims (user role, credit balance), and gain unauthorized admin access in seconds.

---

## Vulnerability

**JWT Token Manipulation (Privilege Escalation)**

Severity: High  
Impact: Complete authentication bypass

---

## Exploitation Steps

1. **Reconnaissance:** nmap + gobuster identified protected routes
2. **Token Extraction:** JWT found in browser storage
3. **Token Decoding:** Payload decoded using jwt.io
4. **Claim Modification:** Changed user→admin, credit→9999999
5. **Token Replacement:** Pasted modified JWT in storage
6. **Admin Access:** Successfully accessed /admin endpoint

---

## Root Cause

- No signature verification on JWT tokens
- Server trusts decoded claims without validation
- Role-based access control relies on unverified token claims
- Backend doesn't re-validate user role on protected routes

---

## Remediation

1. Implement proper JWT signature verification
2. Validate all token claims on backend
3. Check user role in database, not JWT
4. Set token expiration
5. Use HTTPS only

---

**Status:** Vulnerability Confirmed  
**Recommendation:** Implement proper JWT security immediately

---

**Date:** September 25, 2026  
**Tester:** Bishal Kumar Baishya