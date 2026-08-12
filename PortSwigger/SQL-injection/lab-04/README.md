# SQL Injection — UNION Attack: Finding a Column Containing Text

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-finding-columns-with-a-useful-data-type/sql-injection/union-attacks/lab-find-column-containing-text)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

The query results are returned in the application's response, allowing a UNION attack to be used to retrieve data from other tables.

The objective is to perform a SQL injection UNION attack and identify a column that is compatible with string data.

The lab provides a random string that must be made to appear in the query results.

The string provided by the lab was:

```text
Nl9IFs
```

---

## Initial Reconnaissance

I observed that the application uses the same product category filter from the previous SQL injection lab.

The `category` parameter was passed through the URL using the `GET` method.

---

## Interesting Parameter

```text
/filter?category=<ProductCategory>
```

**Method:** `GET`

I used the `category` parameter for SQL injection testing.

---

## Testing

I already determined from the previous lab that the original query returns **3 columns**.

I then used a `UNION SELECT` statement containing `NULL` values and changed the position of a string value to determine which column was compatible with string data.

The testing followed this approach:

```text
' UNION SELECT NULL,'a',NULL--
```

I tested the string value in different column positions.

The purpose was to determine which column could accept and display string data without causing an error.

---

## Exploitation

The lab provided the following random string:

```text
Nl9IFs
```

After identifying the string-compatible column, I replaced the test value:

```text
'a'
```

with the value provided by the lab:

```text
'Nl9IFs'
```

The final UNION injection successfully caused the application to display the required string.

---

## Observation

The application displayed:

```text
Nl9IFs
```

This confirmed that the selected column was compatible with string data.

The lab was successfully solved.

---

## Vulnerability

The `category` parameter is vulnerable to **SQL Injection**.

Because user-controlled input is incorporated into the SQL query, the query can be manipulated using a `UNION SELECT` statement.

---

## Why It Worked

A UNION query combines the results of two SQL queries.

For a UNION attack to work, the injected query must have a compatible number of columns with the original query.

From the previous lab, I determined that the original query returned **3 columns**.

I then tested different column positions to determine which one could accept string data.

Once the string-compatible column was identified, the lab-provided value could be inserted into that column and returned in the application's response.

---

## Impact

The demonstrated impact in this lab was the ability to manipulate the SQL query and make attacker-controlled string data appear in the application's response.

In a real application, a UNION-based SQL injection vulnerability could potentially allow unauthorized retrieval of database information, depending on the database structure and application privileges.

The broader impact was not directly demonstrated in this lab.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should also:

* Use prepared statements for database queries.
* Validate input appropriately.
* Apply least-privilege database permissions.
* Avoid exposing detailed database errors.
* Perform security testing against database-facing parameters.

Input filtering alone should not be relied upon as the primary SQL injection defense.

---

## Tools

* Burp Suite
* Web Browser

---

## Lessons Learned

* How UNION-based SQL injection works.
* Why the number of columns must match when using `UNION`.
* How `NULL` values can be used while testing column compatibility.
* How to identify a column compatible with string data.
* How attacker-controlled data can be returned through a UNION query.
* How the results of one SQL query can be combined with another query.

---

## Key Takeaway

A UNION-based SQL injection attack requires two important pieces of information:

```text
Original query
     ↓
Number of columns
     ↓
Identify compatible data type
     ↓
Place required data in compatible column
     ↓
Observe the returned result
```

In this lab, I used the previously identified **3-column structure**, tested different column positions for string compatibility, and successfully returned the lab-provided value:

```text
Nl9IFs
```
