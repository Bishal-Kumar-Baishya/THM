# Pentest Report

This penetration test of ValenFind dating application identified a critical Local File Inclusion vulnerability that allows unauthenticated attackers to read arbitrary files from the server, including source code and system configuration. Through this vulnerability, we successfully extracted hardcoded API credentials, accessed the entire user database containing 9 user records with passwords, real names, emails, phone numbers, and addresses, and obtained sensitive system files. This represents a complete compromise of application confidentiality and integrity.

**Vulnerability:** Local File Inclusion (LFI) via Path Traversal

**Severity:** Critical

**Affected Endpoint:** `/api/fetch_layout`

**Description:** The endpoint accepts a layout parameter that is directly concatenated into a file path without proper validation. An attacker can use ../ sequences to traverse directories and read arbitrary files.

## Proof of Concept:

```
GET /api/fetch_layout?layout=../../app.py
Response: [full source code of application]
```

## Impact:

- Source code disclosure
- Hardcoded API credentials exposed (ADMIN_API_KEY: "CUPID_MASTER_KEY_2024_XOXO")
- Full database access
- System file access (/etc/passwd, /etc/shadow)
- Complete system compromise

## Remediation:

- Implement strict input validation using an allowlist of permitted layout files
- Use os.path.abspath() and os.path.commonpath() to prevent directory traversal
- Remove hardcoded credentials; use environment variables
- Implement proper access controls on sensitive endpoints


| # | Vulnerability | Severity | Component | Status |
|---|---|---|---|---|
| 1 | Local File Inclusion (Path Traversal) | Critical | /api/fetch_layout | Confirmed |
| 2 | Hardcoded API Credentials | Critical | app.py | Confirmed |
| 3 | Insecure Direct Object Reference (IDOR) | High | /profile/ | Potential |

## Methodology:
- Reconnaissance: Network scanning (nmap), application enumeration
- Vulnerability Analysis: Manual code review, parameter testing
- Exploitation: Path traversal, credential extraction
- Post-Exploitation: Database access, data exfiltration 