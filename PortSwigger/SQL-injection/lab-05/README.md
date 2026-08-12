# SQL Injection — UNION Attack: Retrieving Data from Other Tables

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-using-a-sql-injection-union-attack-to-retrieve-interesting-data/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

The query results are returned in the application's response, allowing a UNION attack to retrieve data from other database tables.

The database contains a table called `users` with the following columns:

```text
username
password
```

The objective is to retrieve the usernames and passwords from the `users` table and use the credentials to log in as the `administrator` user.

---

## Initial Reconnaissance

I observed that the application uses the same product category filter from the previous UNION-based SQL injection labs.

The `category` parameter was supplied through the URL using the `GET` method.

From the previous labs, I already knew that the original query returned **2 columns**, which allowed me to construct a compatible UNION query.

---

## Interesting Parameter

```text
/filter?category=<ProductCategory>
```

**Method:** `GET`

The `category` parameter was used for SQL injection testing.

---

## Testing

I used the techniques learned from the previous UNION SQL injection labs.

The database contained a separate `users` table with:

```text
username
password
```

I constructed a UNION query that selected these two columns from the `users` table:

```sql
' UNION SELECT username,password FROM users--
```

The injected query returned the contents of the `username` and `password` columns in the application's response.

---

## Observation

The application returned the following lab data:

| Username        | Password           |
| --------------- | ------------------ |
| `carlos`        | `[lab credential]` |
| `administrator` | `[lab credential]` |
| `wiener`        | `[lab credential]` |

> **Note:** The credentials above were generated for the PortSwigger lab environment and should not be reused or treated as real credentials.

The `administrator` credentials were then used to authenticate to the application.

---

## Exploitation

The UNION injection successfully retrieved the contents of the `users` table.

I identified the `administrator` account in the returned results and used its lab credentials to log in.

The authentication was successful, completing the lab.

---

## Vulnerability

The `category` parameter is vulnerable to **SQL Injection**.

The application allows user-controlled input to influence the SQL query, making it possible to append a `UNION SELECT` statement and retrieve information from another database table.

---

## Why It Worked

A UNION attack allows the results of another SQL query to be combined with the results of the original query.

The original query returned two columns, so the injected query also needed to return two columns.

The vulnerable application allowed the following database operation to be influenced by user input:

```sql
SELECT username, password
FROM users
```

Because the application returned the query results in its response, the contents of the `users` table became visible.

This allowed the administrator's lab credentials to be recovered and used for authentication.

---

## Impact

The demonstrated impact was significant:

* Unauthorized access to the `users` table.
* Disclosure of usernames.
* Disclosure of passwords.
* Recovery of administrator credentials.
* Authentication as the administrator account.

In a real application, exposing password data could lead to account compromise and potentially broader application compromise, depending on password storage and account privileges.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should also:

* Use prepared statements for database queries.
* Apply least-privilege permissions to database accounts.
* Validate user input appropriately.
* Avoid exposing database errors or query results unnecessarily.
* Store passwords using strong password hashing rather than plaintext.
* Perform security testing against database-facing parameters.

Input filtering alone should not be relied upon as the primary SQL injection defense.

---

## Tools

* Web Browser
* Burp Suite

---

## Lessons Learned

* How UNION-based SQL injection can retrieve data from another table.
* How to use previously identified column counts when constructing a UNION query.
* How database table and column names can be used to retrieve specific information.
* How SQL injection can lead to credential disclosure.
* How credential disclosure can result in authentication bypass or account compromise.
* The importance of parameterized queries and secure password storage.

---

## Key Takeaway

This lab demonstrated the progression from identifying a UNION-compatible SQL injection to retrieving information from another database table.

The attack flow was:

```text
Identify vulnerable parameter
          ↓
Determine column count
          ↓
Identify useful data types
          ↓
Identify target table
          ↓
Retrieve selected columns
          ↓
Recover administrator credentials
          ↓
Authenticate as administrator
```

This lab demonstrated how a SQL injection vulnerability can move from simple query manipulation to **sensitive data disclosure and administrator account compromise**.
