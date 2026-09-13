# Deep Into my Heart - CTF

**Target:** 10.49.186.153
**Difficulty:** easy

## Reconnaissance

### Network Scanning
```bash
sudo nmap -sC -sV 10.49.186.153
```

**Result:** 
- Port 22: SSH
- Port 5000: HTTP (Flask web application)

### Application Discovery
- Visited `http://10.49.186.153:5000`
- Checked the page source, Network tab, and browser console — no forms, hidden elements, or API calls detected.

## Enumeration

Ran Gobuster against the main application URL:
```bash
gobuster dir -u http://10.49.186.153:5000 -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```
which revealed `robots.txt`. After visiting `robots.txt`, it uncovered a hidden path `/cupids_secret_vault/*` with a comment `cupid_arrow_2026!!!`, which could be a password or something else.

After visiting the hidden path, it gives a message
```
Cupid's Secret Vault
You've found the secret vault, but there's more to discover...
``` 

After discovering the vault message, further enumeration was necessary. Ran Gobuster against the discovered vault path:
```bash
gobuster dir -u http://10.49.186.153:5000/cupids_secret_vault/ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```
This revealed an administrator endpoint (Status 200, 2381 bytes), suggesting an admin panel. Visiting this endpoint displayed a login form with Username and Password fields.

## Exploitation

Attempted common default admin usernames: `cupid` (based on application context), then `admin` (standard default). Combined with the password string from robots.txt, the credentials `admin:cupid_arrow_2026!!!` successfully authenticated.

## Key Findings

- Sensitive information (credentials) exposed in robots.txt comments
- Default admin username with weak deployment practices
- Hidden paths provide false sense of security without proper authentication