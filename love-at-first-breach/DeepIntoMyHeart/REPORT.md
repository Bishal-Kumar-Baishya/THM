# Pentest Report: Deep Into my Heart

## Cover Page / Header
- **Title**: Hidden Deep Into my Heart
- **Target:** 10.49.186.153:5000
- **Date:** 13-09-2026
- **Tester:** Bishal Kumar Baishya
- **Difficulty:** Easy

## Executive Summary
During this penetration test, sensitive authentication credentials were discovered in a publicly accessible configuration file (robots.txt). These credentials allowed unauthorized access to the administrative panel, potentially exposing all user data and system functionality to attackers.

## Technical Summary
Nmap reveals open ports which lets to the application in port 5000 (Flask web application). Running gobuster in the main URL reveals the robots.txt which can be accessed by the normal users, where the password of the admin user is exposed.
### Tools used
- Nmap
- Gobuster

## Detailed Findings

### Finding: Sensitive Data Exposure & Inadequate Access Controls in robots.txt
**Severity:** High 

**Description:** The application stores sensitive authentication credentials (admin username and password) in a publicly accessible configuration file (robots.txt) as comments. Additionally, hidden administrative paths are disclosed in the same file, violating the principle of security through obscurity.

**Steps to Reproduce:**
1. Navigate to `http://10.49.186.153:5000/robots.txt`
2. Observe the Disallow path and comment containing credentials
3. Visit the exposed path and use the credentials to authenticate

**Impact:**
- Unauthorized administrative access
- Access to all user data
- Ability to modify/delete content

**Remediation:**
1. Never store credentials or sensitive paths in robots.txt
2. Use environment variables for sensitive configuration
3. Implement proper API key management with rotation
4. Remove version control history containing exposed secrets

## Conclusion
This vulnerability demonstrates a critical failure in secure development practices. Credentials exposed in publicly accessible files represent a complete compromise of administrative security. For any business, such breaches erode customer trust and violate compliance regulations. Regular security assessments and secure development training are essential to prevent similar vulnerabilities in the future.