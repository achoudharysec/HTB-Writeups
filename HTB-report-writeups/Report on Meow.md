# Hack The Box - Meow

| Property | Value |
|----------|-------|
| **Machine** | Meow |
| **Platform** | Hack The Box |
| **Difficulty** | Very Easy |
| **Target IP** | `10.129.169.245` |

---

## Objective

Meow is a **very easy** Linux machine that introduces the basics of the **Telnet** protocol. The objective is to identify the exposed Telnet service, gain remote access, and retrieve the user flag.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.

```text
nmap -A -T4 -Pn <Target-IP>
```

<img width="759" height="215" alt="image" src="https://github.com/user-attachments/assets/2273759c-9e43-444a-8eca-39f140b77530" />


| Port | State | Service | Version |
|------|------|---------|---------|
| 23 | Open | Telnet | Linux telnetd |

### Analysis

The scan identified **Telnet** running on **port 23**. Since Telnet provides remote command-line access and does not encrypt communication, it is considered an insecure service. The next step was to test whether authentication was properly configured.

---

## Exploitation

Connect to the Telnet service.

```text
telnet <Target-IP>
```

<img width="413" height="137" alt="image" src="https://github.com/user-attachments/assets/b3c144c7-30fa-4d44-8c07-90897d7e0c52" />


A connection to the target was successfully established.

---

### Authentication

The Telnet service allowed authentication as the **root** user without requiring a password.

<img width="530" height="139" alt="image" src="https://github.com/user-attachments/assets/946b18af-1c05-40ba-81a9-7ed43c4396b4" />


Successful authentication provided direct **root-level access** to the target system.

---

## Flag Capture

Navigate to the Desktop directory.

```text
cd Desktop
```

List the available files.

```text
ls
```

<img width="124" height="36" alt="image" src="https://github.com/user-attachments/assets/622dd206-af55-4014-a449-2e1b15a15c97" />


Display the contents of the flag file.

```text
cat flag.txt
```

<img width="262" height="30" alt="image" src="https://github.com/user-attachments/assets/af488e78-7949-44e9-a414-d967098dc131" />


The flag was successfully retrieved.

---

## Risk Analysis

### Finding: Unsecured Telnet Service

**Severity:** Critical

**Risk**

The Telnet service permitted direct root authentication without requiring a password, allowing unauthorized users to gain full administrative access.

**Impact**

- Full root-level access
- Unauthorized access to sensitive files
- File modification or deletion
- Installation of malware or backdoors
- Credential theft
- Lateral movement within a network
- Complete system compromise

**Remediation**

- Disable the Telnet service.
- Replace Telnet with **SSH** for secure remote administration.
- Disable direct root login over remote services.
- Enforce strong password authentication.
- Restrict remote access using firewall rules.
- Regularly audit exposed network services.

---

## Root Cause

The system exposed a **Telnet** service that allowed **root authentication without a password**, resulting in unrestricted administrative access to the machine.

---

## Evidence

Successful exploitation was confirmed by:

- Nmap identified an exposed **Telnet** service on **port 23**.
- A Telnet connection was successfully established.
- Authentication as **root** succeeded without a password.
- The `flag.txt` file was successfully accessed and read.

---

## Conclusion

The **Meow** machine demonstrates the risks of exposing insecure remote administration services such as **Telnet**. Because the service allowed direct root authentication without requiring a password, an attacker could immediately gain full administrative control of the system. This machine highlights the importance of disabling legacy services, enforcing secure authentication, and using encrypted alternatives such as **SSH** for remote access.
