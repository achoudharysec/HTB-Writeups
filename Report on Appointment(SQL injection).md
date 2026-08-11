# Hack The Box - Appointment

| Property | Value |
|----------|-------|
| **Machine** | Appointment |
| **Platform** | Hack The Box |
| **Difficulty** | Very Easy |
| **Target IP** | `10.129.93.35` |
| **Operating System** | Linux 5.0–5.14, MikroTik RouterOS 7.2–7.5 (Linux 5.6.3) |

---

## Objective

Appointment is a **very easy** Linux machine that demonstrates a basic **SQL Injection (SQLi)** vulnerability in a web application's authentication mechanism. The objective is to exploit the login form to bypass authentication and retrieve the user flag.

---

## Enumeration

### Nmap Scan

The target was scanned to identify open ports and running services.

| Port | State | Service | Version |
|------|------|---------|---------|
| 80 | Open | HTTP | Apache httpd 2.4.38 |

<img width="335" height="112" alt="image" src="https://github.com/user-attachments/assets/cd17ee54-518b-45b0-a4a1-56931e56e038" />

### Analysis

Only **port 80** was open, indicating that the attack surface was limited to the web application hosted on the Apache HTTP server.

---

## Exploitation

Navigate to the target web application.

```text
http://10.129.93.35
```

or

```text
http://10.129.93.35:80
```

The challenge hint indicated that improper handling of user input could result in part of the SQL query being interpreted as a **comment**.

Use the following credentials:

```text
Username: admin'#
Password: anything
```

<img width="243" height="145" alt="image" src="https://github.com/user-attachments/assets/298c90f5-2239-46ab-a08b-3fdbfadd7971" />


### Explanation

The application constructs the SQL query similar to the following:

```php
$sql = "SELECT * FROM users WHERE username='$username' AND password='$password'";
```

When the username is supplied as:

```text
admin'#
```

the generated query becomes:

```sql
SELECT * FROM users WHERE username='admin'#' AND password='anything';
```

The `#` character starts a **SQL comment**, causing everything after it to be ignored by the database. As a result, the password check is bypassed, allowing authentication as the **admin** user.

---

## Flag Capture

After successfully bypassing authentication, the user flag was displayed.

<img width="486" height="123" alt="image" src="https://github.com/user-attachments/assets/ca40adfb-4237-4f2c-aea6-30d81a15eb1f" />


---

## Risk Analysis

### Finding: SQL Injection Authentication Bypass

**Severity:** Critical

**Risk**

Unsanitized user input allows attackers to manipulate SQL queries and bypass the authentication process without valid credentials.

**Impact**

- Authentication bypass
- Unauthorized access to the application
- Potential exposure of sensitive information
- Possibility of further database compromise depending on user privileges

**Remediation**

- Use **prepared statements (parameterized queries)**.
- Validate and sanitize all user input.
- Avoid constructing SQL queries through string concatenation.
- Apply the principle of least privilege to database accounts.

---

## Conclusion

The **Appointment** machine demonstrates how a simple **SQL Injection** vulnerability can completely bypass an application's authentication mechanism. By injecting a SQL comment into the username field, the password validation was ignored, granting unauthorized access to the application. This machine highlights the importance of secure input handling, parameterized queries, and proper database security practices to prevent SQL injection attacks.
