Machine Name : Sequel
Platform : Hackthebox
difficulty : Very easy
target IP : 10.129.105.250

- **Machine info:** Sequel is a very easy Linux machine that introduces a vulnerable MySQL service misconfigured to allow access without a password. The machine showcases how to enumerate and interact with the database through SQL queries to extract critical information.
- [[Report on Appointment(SQL injection)]] 
### Open Port :

| Port | State | Service | Version                         |
| ---- | ----- | ------- | ------------------------------- |
| 3306 | Open  | mysql?  | 5.5.5-10.3.27-MariaDB-0+deb10u1 |
### Enumeration :
**nmap :**
![[Pasted image 20260718174913.png]]

### Exploitation :

- A **remote connection** to the MariaDB service :
![[Pasted image 20260719213454.png]]
successfully established using the privileged `root` account without providing a valid password.
```
mysql -h 10.129.107.117 -u root -p --skip-ssl -A
```

- selected htb database using :
```
USE htb;
```

- queried config table :
```
SELECT * FROM config;
```

### Risk Severity: Critical

> The MariaDB service exposed on TCP port **3306** was configured to permit remote access to the `root` database account without requiring a valid password. This allows an unauthenticated attacker with network access to connect to the database with highly privileged permissions and access sensitive information stored within it.

### Impact

> Successful exploitation of this misconfiguration allows an unauthorized attacker to access and enumerate the database server. Depending on the privileges assigned to the database account, an attacker may:
> 
> - Access sensitive information stored within databases.
> - Enumerate database structures, tables, and configuration data.
> - Read application or user information stored in accessible tables.
> - Modify or delete database records if sufficient privileges are granted.
> - Disrupt applications that depend on the affected database.
> - Extract sensitive configuration information or credentials stored within database tables.
> 
> During testing, unauthorized access to the `htb` database was successfully demonstrated, and sensitive configuration data, including the challenge flag, was retrieved from the `config` table.

This matches the evidence on pages 2–3, where `SELECT * FROM config;` successfully exposes all configuration values.

### Root Cause

> The primary root cause was an insecure MariaDB configuration that allowed the privileged `root` database account to authenticate remotely without a valid password. Additionally, the database service was exposed over the network on TCP port **3306**, allowing an unauthenticated remote user to directly interact with the database server.

### Remediation

> - Configure a strong, unique password for all privileged MariaDB accounts, especially `root`.
> - Disable remote login for the MariaDB `root` account unless explicitly required.
> - Create dedicated database users with only the minimum privileges required for their intended purpose.
> - Restrict TCP port **3306** using firewall rules so that only trusted systems can access the database.
> - Bind MariaDB to localhost when remote database access is unnecessary.
> - Regularly audit database accounts, authentication settings, and granted privileges.
> - Avoid storing sensitive information such as credentials or secrets in unnecessarily accessible database tables.

### Evidence

> The following evidence confirms the security misconfiguration:
> 
> - Nmap identified **MariaDB 10.3.27** exposed on TCP port **3306**.
> - A remote connection to MariaDB was successfully established using the privileged `root` account without providing a valid password.
> - Database enumeration revealed the `htb` database.
> - The `htb` database contained the `config` and `users` tables.
> - The `config` table was successfully queried without authorization.
> - Sensitive configuration information and the challenge flag were successfully retrieved.

Your screenshots/query output already demonstrate most of this clearly.

### Conclusion

> The target database server was successfully accessed due to an insecure MariaDB configuration that permitted remote authentication to the privileged `root` account without a valid password. This allowed unauthorized enumeration of the database server and access to the `htb` database, from which sensitive configuration information was retrieved. The misconfiguration could expose sensitive application data to unauthorized users and potentially allow modification or deletion of database contents depending on the privileges granted to the compromised account. Restricting remote database access, enforcing strong authentication, and applying the principle of least privilege are required to prevent unauthorized access.