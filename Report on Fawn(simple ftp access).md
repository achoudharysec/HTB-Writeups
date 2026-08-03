Machine name : Fawn
Platform : Hackthebox
difficulty : very easy 
target IP : 10.129.84.76
OS : Unix 

**Description :** Initial reconnaissance identified as exposed File Transfer Protocol (FTP/21) service. 

**Open Ports :** 21 , state : open , service : ftp , version : vsftpd 3.0.3
- default network port used by the File Transfer Protocol (FTP) to establish control connections between a client and a server
```
nmap -sV 10.129.84.76
```

![[Pasted image 20260708191820.png]]

**Exploitation :** of open port 21 
```
ftp -a 10.129.84.76
```
- this command provides you the login into the system through anonymous user , only if it is configured to give access anonymously. 
![[Pasted image 20260708194454.png]]
- enumerate files with the `ls`.
![[Pasted image 20260708194616.png]]
- download the file from the targets system using `get`.
![[Pasted image 20260708200140.png|697]]
- after download give the `bye` command.
![[Pasted image 20260708200916.png]]
- the using the `ls` in your system , open the flag.txt using the `cat`.
```
cat flag.txt
```
![[Pasted image 20260708201126.png]]
