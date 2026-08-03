Machine name : Appointment
Platform : Hackthebox
difficulty : very easy 
target IP : 10.129.93.35
OS : Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)

- Appointment is a very easy Linux machine which showcases beginner SQL Injection techniques against an SQL database enabled web application.

#### Open Ports : 
 -   80 , status : open  , service:  http   ,version: Apache httpd 2.4.38
![[Pasted image 20260712232803.png]]

#### Exploit : 
- Go to the http: 
```
http://10.129.93.35  or http://10.129.93.35:80
```

- As asked in the question (ie.If user input is not handled carefully, it could be interpreted as a **comment**) , so enter the # at the end of the username.
```
Username : admin'# 
Password : {anything}
```
![[Pasted image 20260712234105.png]]

- what it does is that, the **PHP & SQL** using the authentication like :
`$sql="SELECT * FROM users WHERE username='$username' AND password='$password'";`
and the (#) is treated as a comment so if we put the username as `admin'#` , rest of the code after the # will be treated as the comment , which is SQL injection.

- Flag is captured :
![[Pasted image 20260712234038.png]]

#### Risk assessment :
**Risk severity** : critical
- The vulnerability allows authentication bypass without valid credentials. An attacker can gain unauthorized access and potentially compromise sensitive data depending on the privileges of the targeted account.
**Impact:**
- Reading all files
- Modifying files
- Deleting files

#### Conclusion : 
The login page was successfully breached through an SQL injection where you can modify the query (the $sql variable) through the log-in form on the web page to make the query do something that is not supposed to do - bypass the log-in altogether!.