01-SQLi-Data-Extraction.md
# SQL Injection – Retrieval of Hidden Data (Proof Included)

This assessment was performed on a deliberately vulnerable application from the PortSwigger using Burp Suite. The objective was to identify whether user-controlled input within the application could influence backend SQL queries and to exploit this behavior to bypass filtering mechanisms and retrieve hidden or unreleased data.

The testing environment was configured by routing all browser traffic through Burp Suite via the local proxy (`127.0.0.1:8080`). After launching the application, initial reconnaissance involved interacting with the user interface, specifically by selecting product categories. This action generated HTTP requests that were captured and analyzed in Burp Suite. One request of interest was identified:

```http id="9x7k2q"
GET /filter?category=Food+%26+Drink HTTP/2
```

The presence of the `category` parameter indicated that user input was directly controlling product filtering logic. Based on standard application behavior, it was inferred that this parameter was embedded into a backend SQL query, likely similar to:

```sql id="3kz9dm"
SELECT * FROM products WHERE category = 'Food & Drink';
```

Given that this input was user-controlled, it represented a potential SQL injection point if not properly sanitized. To validate this, the request was sent to Burp Suite Repeater for controlled testing. The `category` parameter was modified using a boolean-based SQL injection payload:

```sql id="7f2wqs"
' OR 1=1--
```

This produced the following manipulated request:

```http id="p4z8nh"
GET /filter?category='+OR+1=1-- HTTP/2
```

Upon execution, the server returned a significantly expanded dataset, displaying all available products rather than only those within the selected category. This confirmed that the injection successfully altered the backend query logic. Technically, the payload transformed the SQL condition into:

```sql id="c2v6rp"
WHERE category = '' OR 1=1--'
```

Since the condition `OR 1=1` always evaluates to true, the query returns all records, while the comment sequence `--` truncates the remainder of the original query to prevent syntax errors. As a result, the intended filtering mechanism was completely bypassed, exposing all data within the table.

To ensure the exploit was valid in a real application context, the same payload was applied using Burp Suite Interceptor. Interception was enabled, a category selection was triggered in the browser, and the outgoing request was modified in transit before being forwarded to the server. The browser subsequently displayed all products across multiple categories, confirming that the vulnerability is exploitable within normal user interaction and not limited to controlled testing environments.

Proof of exploitation is established through multiple observations: under normal conditions, the request `/filter?category=Food+%26+Drink` returns only products within that category, whereas the injected request `/filter?category='+OR+1=1--` results in the display of all products, including those outside the selected category and previously hidden entries. Additionally, the application reflects the injected payload in the response and updates the lab status to “Solved,” confirming successful exploitation.

The impact of this vulnerability is significant, as it allows an attacker to bypass application-level restrictions and gain unauthorized access to data that should not be exposed. In real-world scenarios, such vulnerabilities can lead to full database compromise, including the extraction of sensitive information such as user credentials or internal business data.

The root cause of the issue lies in the unsafe handling of user input within SQL queries. Specifically, the application fails to implement parameterized queries or proper input validation, allowing direct manipulation of query logic. To remediate this vulnerability, developers should use prepared statements, enforce strict input validation, and avoid constructing SQL queries through direct string concatenation.

In conclusion, this exercise demonstrates a fundamental SQL injection vulnerability and highlights the importance of secure coding practices. It reinforces key penetration testing principles, including traffic analysis, hypothesis-driven testing, controlled exploitation, and validation within real application workflows, all of which are essential for identifying and assessing real-world security flaws.
