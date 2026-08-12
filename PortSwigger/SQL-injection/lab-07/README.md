# SQL Injection — Querying the Database Type and Version on MySQL and Microsoft

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-examining-the-database-in-sql-injection-attacks/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

A UNION attack can be used to retrieve the results of an injected SQL query.

The objective is to display the database version string.

---

## Initial Reconnaissance

I observed that the application uses the product category filter through an HTTP `GET` request.

The category parameter was vulnerable to SQL injection testing.

However, the approach used in the previous labs did not immediately work in this lab, so I inspected and modified the HTTP request using Burp Suite.

---

## Interesting Parameter

```text id="i2q1vo"
/filter?category=<ProductCategory>
```

**Method:** `GET`

The request was captured and modified using **Burp Suite Repeater**.

---

## Testing

I first tested the SQL injection using techniques from the previous labs.

### ORDER BY

I tested the number of columns using `ORDER BY`.

### UNION SELECT NULL

I then tested UNION queries using `NULL` values.

I also tested:

```sql id="4r7eis"
' UNION SELECT 'abc','def'#
```

The request required modification directly in Burp Suite because the normal browser request did not produce the expected result.

---

## Exploitation

After modifying the HTTP request in Burp Suite, I used the MySQL/Microsoft-specific version variable:

```sql id="fyscwb"
@@version
```

The payload used was:

```text id="6h5v1q"
'+UNION+SELECT+@@version,+NULL#
```

The injected query retrieved the database version information and returned it in the application's response.

---

## Observation

The application displayed a database software version string.

This confirmed that the SQL injection could be used to query information about the underlying database software.

---

## Vulnerability

The product category parameter is vulnerable to **SQL Injection**.

The application allows user-controlled input to influence the SQL query, making it possible to append a UNION query and retrieve information from the database.

---

## Why It Worked

The injection worked because the application did not safely separate user input from SQL syntax.

The `@@version` variable is supported by MySQL and Microsoft SQL Server and can return information about the database version.

The UNION query combined the injected query with the application's original query, allowing the database version string to be returned in the application's response.

---

## Impact

The demonstrated impact was:

* Ability to execute an additional SQL query through the vulnerable parameter.
* Disclosure of the underlying database software version.
* Information gathering about the database environment.

Database version information can help an attacker identify the database technology and potentially research database-specific vulnerabilities or attack techniques.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should also:

* Use prepared statements for database queries.
* Apply least-privilege permissions to database accounts.
* Validate user input appropriately.
* Avoid unnecessarily exposing database information.
* Avoid displaying detailed database errors to users.
* Perform regular security testing against database-facing parameters.

Input filtering alone should not be relied upon as the primary SQL injection defense.

---

## Tools

* Burp Suite
* Burp Repeater
* Web Browser

---

## Lessons Learned

* How SQL injection can be used to identify database information.
* How UNION attacks can retrieve the result of an injected query.
* How database-specific functions and variables can be useful during SQL injection testing.
* How `@@version` can be used with MySQL and Microsoft SQL Server.
* How to modify and test HTTP requests using Burp Suite Repeater.
* Why understanding the database type is useful during SQL injection testing.

---

## Key Takeaway

This lab demonstrated that SQL injection can be used not only to retrieve application data but also to gather information about the underlying database environment.

The attack flow was:

```text
Identify vulnerable parameter
          ↓
Determine UNION structure
          ↓
Modify request in Burp Repeater
          ↓
Use database-specific version query
          ↓
Retrieve database version
          ↓
Identify database technology
```
