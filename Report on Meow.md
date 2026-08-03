Machine name : Meow
Platform : Hackthebox
difficulty : very easy 
target IP : 10.129.169.245

Description :  This machine is Meow , Provided by the HackTheBox and this machine was exposing the 23 Port  Open which allowed access to the root authority , making the system vulnerable.

**Open Port** : 23 , state : Open ,  Service : telnet , Version : Linux telnetd 
 a 23 telnet port allows you to you to remotely log in to another computer and control it through a command-line interface (CLI).)
 ```
 nmap -A -T4 -Pn {target ip}
 ```
 ![[Pasted image 20260708161612.png|689]]
#### Exploitation of port 23 : 

```
telnet {target ip}
```
The Telnet service was used to obtain remote access to the target system.
![[Pasted image 20260708163524.png]]
gained access to the system

- **Allowing** easy access to the root authority with blank required.
![[Pasted image 20260708162459.png]]
gained the privilege of root authority.

- go to the desktop with cd Desktop.
```
cd Desktop
```

- Enumerate files with `ls`.
![[Pasted image 20260708162924.png|124]]

- check the text inside the `flag.txt`
using command :
```
cat flag.txt
```
then copy the text in it and then submit the flag in hack the box.
![[Pasted image 20260708163242.png]]

##### Risk Assessment:
**Severity:**  Critical
**Impact:**
- Reading all files
- Modifying files
- Deleting files
- Installing malware
- Creating new users
- Backdoor installation
- Credential theft
- Pivoting into internal networks
- Ransomware deployment
##### **conclusion  :**
The machine was successfully compromised through an exposed Telnet service that permits authentication with the root account without a password. The attack resulted in SYSTEM privileges and credential extraction,data theft, demonstrating the importance of timely patch management and secure SMB configuration.