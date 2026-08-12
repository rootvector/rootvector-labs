# SQL Injection — UNION Attack: Retrieving Multiple Values in a Single Column

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-retrieving-multiple-values-within-a-single-column/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

The query results are returned in the application's response, allowing a UNION attack to retrieve data from other database tables.

The database contains a table called `users` with the following columns:

```text
username
password
```

The objective is to retrieve all usernames and passwords and use the information to log in as the `administrator` user.

---

## Initial Reconnaissance

I observed that the application uses the same product category filter from the previous UNION-based SQL injection labs.

The `category` parameter is supplied through the URL using the `GET` method.

From the previous labs, I knew the required column structure for the UNION query.

---

## Interesting Parameter

```text
/filter?category=<ProductCategory>
```

**Method:** `GET`

The `category` parameter was used for SQL injection testing.

---

## Testing

I used the UNION SQL injection technique from the previous labs.

In this lab, the challenge was different because the `username` and `password` values needed to be retrieved through a **single column**.

I concatenated the `username` and `password` values into one string using the `||` concatenation operator and separated them with the `~` character.

The payload used was:

```sql
' UNION SELECT NULL,username || '~' || password FROM users--
```

Conceptually, the query combines:

```text
username
   +
~
   +
password
```

into a single returned value.

---

## Observation

The application returned the following values:

| Retrieved Value                  |
| -------------------------------- |
| `carlos~[lab credential]`        |
| `administrator~[lab credential]` |
| `wiener~[lab credential]`        |

The `administrator` username and its corresponding lab password were visible in the results.

---

## Exploitation

After retrieving the usernames and passwords from the `users` table, I identified the `administrator` account.

I used the corresponding lab credentials to log in to the application as the administrator.

The login was successful and the lab was solved.

---

## Vulnerability

The `category` parameter is vulnerable to **SQL Injection**.

The application allows user-controlled input to influence the SQL query, making it possible to append a UNION query and retrieve information from another database table.

---

## Why It Worked

In the previous UNION lab, the username and password were returned through separate columns.

In this lab, the required information needed to be returned through a **single column**.

The SQL concatenation operator:

```sql
||
```

was used to combine multiple values into one string.

Conceptually:

```text
username || '~' || password
```

produces:

```text
username~password
```

This allowed both values to be returned through the same column of the UNION result.

---

## Impact

The demonstrated impact was:

* Unauthorized access to the `users` table.
* Disclosure of usernames.
* Disclosure of passwords.
* Recovery of administrator credentials.
* Authentication as the administrator account.

In a real application, this type of vulnerability could lead to account compromise and potentially broader unauthorized access.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should also:

* Use prepared statements for database queries.
* Apply least-privilege permissions to database accounts.
* Validate user input appropriately.
* Avoid exposing database query results unnecessarily.
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
* How to retrieve multiple database values through a single column.
* How SQL string concatenation works using `||`.
* How separators can make concatenated values easier to distinguish.
* How SQL injection can result in credential disclosure.
* How credential disclosure can lead to administrator account compromise.

---

## Key Takeaway

This lab demonstrated how multiple database values can be combined into a single output column when a UNION attack cannot directly return each value in a separate column.

The attack flow was:

```text
Identify vulnerable parameter
          ↓
Determine UNION column structure
          ↓
Identify available output column
          ↓
Concatenate username + separator + password
          ↓
Retrieve credentials
          ↓
Identify administrator account
          ↓
Authenticate as administrator
```
