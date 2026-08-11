# Hack The Box - Fawn

| Property | Value |
|----------|-------|
| **Machine** | Fawn |
| **Platform** | Hack The Box |
| **Difficulty** | Very Easy |
| **Target IP** | `10.129.84.76` |
| **Operating System** | Unix |

---

## Objective

Fawn is a **very easy** machine that introduces the basics of the **File Transfer Protocol (FTP)**. The objective is to identify an exposed FTP service, determine whether anonymous authentication is enabled, and retrieve the flag from the target system.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.

```text
nmap -sV 10.129.84.76
```

<img width="527" height="130" alt="image" src="https://github.com/user-attachments/assets/258faa44-825c-413f-a3f9-12cef4087c82" />


| Port | State | Service | Version |
|------|------|---------|---------|
| 21 | Open | FTP | vsftpd 3.0.3 |

### Analysis

The scan identified an FTP service running **vsftpd 3.0.3** on **port 21**. Since FTP commonly supports anonymous authentication, the service was tested for anonymous access.

---

## Exploitation

Connect to the FTP server using anonymous authentication.

```text
ftp -a 10.129.84.76
```

The `-a` option attempts to log in anonymously if the FTP server allows anonymous access.

<img width="306" height="114" alt="image" src="https://github.com/user-attachments/assets/ff2cea24-b4ba-4e95-a256-cc71ce92a5a0" />


The login was successful, confirming that anonymous FTP access was enabled.

---

## Enumeration

List the available files on the FTP server.

```text
ls
```

<img width="526" height="82" alt="image" src="https://github.com/user-attachments/assets/8b06ca8a-9217-406e-a1f9-84ea2860c684" />


A file named **flag.txt** was identified.

---

## File Retrieval

Download the file from the target.

```text
get flag.txt
```

<img width="846" height="114" alt="image" src="https://github.com/user-attachments/assets/4d404a1b-84bf-4b8c-be0d-2416cb02c992" />


Exit the FTP session.
```
bye
```

---

## Flag Capture

Verify that the file was downloaded successfully.

```text
ls
```

Display the contents of the file.

```text
cat flag.txt
```

<img width="373" height="126" alt="image" src="https://github.com/user-attachments/assets/f29cebd3-3dc1-468b-b0ba-d7190af6bbf7" />


The flag was successfully retrieved from the FTP server.

---

## Risk Analysis

### Finding: Anonymous FTP Access

**Severity:** Medium

**Risk**

The FTP server permits anonymous authentication, allowing unauthenticated users to access files stored on the server.

**Impact**

- Unauthorized access to publicly exposed files.
- Disclosure of sensitive information.
- Increased attack surface for further enumeration.

**Remediation**

- Disable anonymous FTP access unless explicitly required.
- Require authenticated user accounts for FTP access.
- Restrict file permissions based on the principle of least privilege.
- Replace FTP with secure alternatives such as **SFTP** where appropriate.

---

## Root Cause

The FTP service was configured to allow **anonymous authentication**, enabling any user to access and download files without valid credentials.

---

## Evidence

- Nmap identified an FTP service running on **port 21**.
- Anonymous login was successful.
- The file **flag.txt** was listed and downloaded.
- The flag was successfully read from the downloaded file.

---

## Conclusion

The **Fawn** machine demonstrates the security risks associated with **anonymous FTP access**. Because the FTP server allowed unauthenticated users to log in, sensitive files could be enumerated and downloaded without valid credentials. This machine highlights the importance of disabling anonymous FTP access and implementing proper authentication mechanisms to protect exposed services.
