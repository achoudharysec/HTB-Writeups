# Hack The Box - Nibbels
> **Machine:** Nibbels
>
> **Platform:** Hack The Box
>
> **Difficulty:** Easy
>
> **Target IP:** `10.10.10.75`

---
**Objective :** This version preserves the evidence and attack path from the uploaded report while adding clear explanations, methodology, findings, command context, and a concise
lessons-learned section.

---
## 1. Enumeration:
### nmap :
An initial full-port Nmap scan was performed to identify exposed services.
```
nmap -A -T4 -p- 10.10.10.75
```

The scan identified two relevant services:

| Port     | Service | Version       |
| -------- | ------- | ------------- |
| `22/tcp` | SSH     | OpenSSH 7.2p2 |
| `80/tcp` | HTTP    | Apache 2.4.18 |
![](z(Asets)/Pasted%20image%2020260811172226.png)

Port `80` became the primary focus because it exposed a web application.
## 2. Website Enumeration:
### Website:

Navigating to:
```
http://10.10.10.75/
```
revealed a website running **Nibbleblog**.
![618](../../../../z%20(Attachments)/Pasted%20image%2020260806230103.png)
The page title identified the site as:
> **Nibbles Yum yum**

Since the web application was the primary attack surface, further content enumeration was performed.

## 3. Web & Directory Enumeration:
### Searchsploit:

[Searchsploit](../../../../tools/Exploitation/Searchsploit.md) to search the local Exploit Database index for known Nibbleblog vulnerabilities.
![](../../../../z%20(Attachments)/Pasted%20image%2020260807143347.png)
found the `Arbitrary file uplo` in it with remote code execution.

### Metasploit Module Research:

The corresponding [[../../../../tools/Exploitation/Metasploit]]module was located with a Nibbleblog search.

![](../../../../z%20(Attachments)/Pasted%20image%2020260807154858.png)

check the info:
```
info
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260807155007.png)
The module information indicated that the upload vulnerability is exploitable by an authenticated attacker.

Therefore, the next objective was to locate the administrative login endpoint and obtain valid credentials.

### Directory / Content Enumeration:

[[../../../../tools/Recon/Gobuster]] was used against the Nibbleblog installation to discover hidden web content.

The original report used the SecLists common web-content wordlist.

```
gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

The enumeration located the administrative login page:

```
http://10.10.10.75/nibbleblog/admin.php
```

![613](../../../../z%20(Attachments)/Pasted%20image%2020260807191416.png)

## 3. Credential Discovery:

The discovered `admin.php` endpoint presented a Nibbleblog administration login.
you can use [Brute force using burp-suit](../../../../Y%20(extra%20notes)/Brute%20force%20using%20burp-suit.md):

-  the credentials were found-able in the content-page.
![](../../../../z%20(Attachments)/Pasted%20image%2020260807194649.png)

The report captured the following application information:
```
<notification_email_to type="string">admin@nibbles.com</notification_email_to>
<seo_site_title type="string">Nibbles - Yum yum</seo_site_title>
```

From this information, the report identified:
```
Username: admin
Password: nibbles
```
## 4. Exploitation & Initial Access:

After authenticating to Nibbleblog, the report navigated to the **Plugins** area and identified the **My Image** plugin, which allowed the administrator to upload an image.

This upload functionality formed the practical exploitation point for the known arbitrary file-upload vulnerability.
![](../../../../z%20(Attachments)/Pasted%20image%2020260807224757.png)

### Metasploit Exploitation:

The Metasploit module was then configured with the target web path, valid application credentials, and a reverse connection listener.

![](../../../../z%20(Attachments)/Pasted%20image%2020260807231503.png)
the shell was successfully acquired.

## 5. Privilege Escalation:
### Check Sudo Permissions:

run:
```
sudo -l
```
It lists **which commands the current user is allowed to run using `sudo`**.

The critical entry allowed the user to execute the following path as root:
![](../../../../z%20(Attachments)/Pasted%20image%2020260808182923.png)

The home directory and observed that `monitor.sh` did not exist.:
![](../../../../z%20(Attachments)/Pasted%20image%2020260808185839.png)

### Create the Missing Script:

![](../../../../z%20(Attachments)/Pasted%20image%2020260808190102.png)
`bash -i` the `-i` stands for **interactive**.

It launches **an interactive Bash shell** where you can type commands and immediately receive responses.

## 6. Root Access & Final Result:
### Make the Script Executable:
- **upgrade the permissions,** of the file as `-rwx`
```
chmod +x monitor.sh ls -la
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260808190754.png)

### Execute Through Sudo:

execute the attacker-controlled script through the sudo rule:
```
sudo /home/nibbler/personal/stuff/monitor.sh
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260808191734.png)successful privilege escalation from the `nibbler` account to root.

## Risk Analysis:

The overall risk is **Critical** because the vulnerabilities could be chained to gain complete control of the system.

- **Nibbleblog file upload — Critical:** Allowed authenticated attackers to achieve remote code execution.
- **Credential exposure — High:** Exposed information helped obtain administrative access.
- **Sudo misconfiguration — Critical:** The `nibbler` user could execute a user-controlled script as root, leading to full privilege escalation.

**Overall:** The attack path progressed from **web access → remote shell → root access**, resulting in complete system compromise.

## Conclusion:

The Nibbles assessment successfully demonstrated a complete compromise through web enumeration, credential discovery, exploitation of the Nibbleblog file-upload vulnerability, and an insecure sudo configuration.

The main lesson is that **small security weaknesses can be chained together to achieve full system compromise**. Updating vulnerable software, protecting credentials, and properly securing sudo permissions would significantly reduce the risk.