# Hack The Box - Sequel

| Property | Value |
|----------|-------|
| **Machine** | Sequel |
| **Platform** | Hack The Box |
| **Difficulty** | Very Easy |
| **Target IP** | `10.129.105.250` |
| **Operating System** | Linux |

---

## Objective

Sequel is a **very easy** Linux machine that demonstrates the risks of exposing a **MariaDB** service with insecure authentication. The objective is to enumerate the database service, authenticate using the misconfigured root account, and retrieve sensitive information stored within the database.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.

<img width="528" height="194" alt="image" src="https://github.com/user-attachments/assets/d64cacf2-c173-494f-820a-7de1612d7f8e" />


| Port | State | Service | Version |
|------|------|---------|---------|
| 3306 | Open | MariaDB | 10.3.27-MariaDB-0+deb10u1 |

### Analysis

The scan identified a **MariaDB** service exposed on **TCP port 3306**. Since database services should not normally allow unrestricted remote access, authentication was tested using the default administrative account.

---

## Exploitation

### Remote Database Access

Connect to the MariaDB server.

```text
mysql -h <Target-IP> -u root -p --skip-ssl -A
```

<img width="638" height="425" alt="image" src="https://github.com/user-attachments/assets/4884a929-b63e-40de-a45d-3922eb3db6d9" />


A remote connection was successfully established using the privileged **root** account **without providing a valid password**, confirming an insecure database configuration.

---

### Database Enumeration

Select the target database.

```sql
USE htb;
```

Query the configuration table.

```sql
SELECT * FROM config;
```

The query successfully returned sensitive configuration information, including the challenge flag.

---

## Risk Analysis

### Finding: Insecure MariaDB Authentication

**Severity:** Critical

**Risk**

The MariaDB service permitted remote authentication to the privileged **root** account without requiring a valid password, allowing unauthorized users to gain administrative access to the database server.

**Impact**

- Unauthorized database access
- Enumeration of databases and tables
- Disclosure of sensitive configuration data
- Access to application information
- Modification or deletion of database records
- Potential disruption of dependent applications

---

## Root Cause

The MariaDB server was **misconfigured** to allow remote authentication for the **root** account without enforcing a valid password. In addition, the database service was exposed on **TCP port 3306**, making it directly accessible over the network.

---

## Evidence

Successful exploitation was confirmed by:

- Nmap identified **MariaDB 10.3.27** running on **TCP port 3306**.
- Remote authentication as **root** succeeded without a valid password.
- The **htb** database was successfully enumerated.
- The **config** table was queried successfully.
- Sensitive configuration data, including the challenge flag, was retrieved.

---

## Remediation

- Configure strong, unique passwords for all privileged database accounts.
- Disable remote login for the **root** account unless explicitly required.
- Create dedicated database users with the minimum required privileges.
- Restrict access to **TCP port 3306** using firewall rules.
- Bind MariaDB to **localhost** when remote access is unnecessary.
- Regularly audit database users, authentication methods, and granted privileges.
- Avoid storing sensitive information in publicly accessible database tables.

---

## Conclusion

The **Sequel** machine demonstrates how an insecure **MariaDB** configuration can expose critical database services to unauthorized users. Because the privileged **root** account permitted remote authentication without a valid password, the database could be fully enumerated and sensitive configuration data was retrieved. This machine highlights the importance of enforcing strong authentication, restricting remote database access, and applying the principle of least privilege to database accounts.
