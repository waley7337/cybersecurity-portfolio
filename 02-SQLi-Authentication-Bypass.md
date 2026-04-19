# SQL Injection in Login Functionality (Authentication Bypass)
Here’s your **final GitHub-ready report in one clean flow with embedded proof** — professional, realistic, and portfolio-level:

---

# 🔐 SQL Injection – Authentication Bypass

A SQL injection vulnerability was identified in the login functionality of a web application hosted on the PortSwigger Web Security Academy. The vulnerability allows an attacker to bypass authentication controls and gain unauthorized access to privileged accounts, including the administrator, without valid credentials.

During testing, the login request was intercepted using Burp Suite. The application sends user credentials via a POST request to the `/login` endpoint, where both `username` and `password` parameters are included in the request body along with a CSRF token:

```http
POST /login HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded

csrf=<token>&username=administrator&password=administrator
```

Analysis of the request indicated that user-supplied input is directly incorporated into a backend SQL query without proper sanitization or parameterization. To test for injection, the `username` parameter was modified by appending a SQL comment sequence, effectively altering the logic of the query:

```http
csrf=<token>&username=administrator'--&password=administrator
```

This payload terminates the original string context and comments out the remainder of the SQL query, including the password verification condition. The resulting backend query can be logically interpreted as:

```sql
SELECT * FROM users 
WHERE username = 'administrator'-- ' AND password = 'administrator'
```

Because the password check is ignored, the application authenticates the request solely based on the username. This behavior was confirmed when the server granted access as the `administrator` user without requiring a valid password, successfully demonstrating a full authentication bypass.

### 📸 Proof of Exploitation

The following request was captured and modified in Burp Suite, showing the injected payload:

```http
csrf=P6Sk...&username=administrator'--&password=administrator
```

Upon forwarding this request, the application responded with a valid authenticated session, confirming that the attacker was logged in as the administrator and that the lab was successfully solved.

---

This vulnerability has critical security implications, as it allows complete account takeover and unrestricted access to sensitive functionality. In a real-world environment, such a flaw could lead to full system compromise, data exfiltration, and privilege escalation.

The root cause of the issue is the unsafe construction of SQL queries through direct concatenation of user input, combined with the absence of parameterized queries or prepared statements. This allows attackers to manipulate the structure of database queries and bypass intended security controls.

To remediate this vulnerability, it is essential to implement parameterized queries to ensure that user input is treated strictly as data rather than executable code. Additional defenses include strict input validation, avoiding raw SQL query construction, and strengthening authentication mechanisms through controls such as rate limiting and multi-factor authentication.

This assessment demonstrates how a simple injection payload can compromise an entire authentication system, highlighting the importance of secure coding practices and proper input handling in web applications.
