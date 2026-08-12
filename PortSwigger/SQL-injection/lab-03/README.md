# SQL Injection — UNION Attack: Determining the Number of Columns Returned by the Query

[Lab URL](PASTE_LAB_URL_HERE)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

The query results are returned in the application's response, allowing a UNION attack to be used to retrieve data from other tables.

The objective of this lab is to determine the number of columns returned by the original SQL query by performing a SQL injection UNION attack that returns an additional row containing `NULL` values.

---

## Initial Reconnaissance

I observed that the application uses a product category filter and that the selected category is passed through the URL.

---

## Interesting Parameter

```text
/filter?category=<ProductCategory>
```

**Method:** `GET`

The `category` parameter was selected for SQL injection testing.

---

## Testing

I used Burp Suite to modify the `category` parameter and tested two techniques:

* `ORDER BY`
* `UNION SELECT`

### 1. ORDER BY Testing

I incremented the column number to determine how many columns were returned by the original query.

The tests followed this pattern:

```text
'+ORDER+BY+1--
'+ORDER+BY+2--
'+ORDER+BY+3--
```

The application continued to respond successfully up to column `3`.

This indicated that the original query returned **3 columns**.

---

### 2. UNION SELECT Testing

I then tested the number of columns using `UNION SELECT` and `NULL` values.

The tests followed this pattern:

```text
' UNION SELECT NULL--
```

Then:

```text
' UNION SELECT NULL,NULL--
```

And finally:

```text
' UNION SELECT NULL,NULL,NULL--
```

The three-column version was accepted by the application, confirming the result from the `ORDER BY` testing.

---

## Observation

The original SQL query returns:

```text
3 columns
```

The lab was successfully solved by determining the correct number of columns.

No data extraction was performed because this lab only required determining the column count.

---

## Vulnerability

The `category` parameter is vulnerable to **SQL Injection**.

The input can influence the SQL query, allowing SQL syntax such as `ORDER BY` and `UNION SELECT` to alter the query's behavior.

---

## Exploitation

I used two techniques to determine the number of columns returned by the SQL query:

1. Incrementing `ORDER BY` values.
2. Supplying an increasing number of `NULL` values with `UNION SELECT`.

Both techniques indicated that the query returns **3 columns**.

---

## Why It Worked

The application incorporates user-controlled input into the SQL query without safely separating SQL syntax from user data.

Because the injected input is interpreted as part of the SQL statement, I was able to test the structure of the underlying query.

The `ORDER BY` technique helped identify the maximum valid column index, while the `UNION SELECT` technique confirmed the number of columns required for a compatible UNION query.

---

## Impact

In this lab, the demonstrated impact was the ability to manipulate the SQL query and determine its column structure.

This information is useful for constructing subsequent UNION-based SQL injection attacks.

The lab itself did **not** demonstrate extraction of sensitive database information, so I am not claiming that impact here.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should also:

* Use prepared statements for database queries.
* Validate user input appropriately.
* Apply least-privilege permissions to database accounts.
* Avoid exposing detailed database errors.
* Perform security testing on database-facing parameters.

Input filtering alone should not be relied upon as the primary SQL injection defense.

---

## Tools

* Burp Suite
* Web Browser

---

## Lessons Learned

* How UNION-based SQL injection works.
* How to determine the number of columns returned by a query.
* How `ORDER BY` can be used to identify the column count.
* How `UNION SELECT NULL` can confirm the column count.
* Why the number of columns must match when using `UNION`.
* How understanding the SQL query structure is important before attempting data extraction.

---

## Key Takeaway

Before performing a UNION-based SQL injection attack, it is necessary to determine the number of columns returned by the original query.

In this lab, both `ORDER BY` testing and `UNION SELECT NULL` testing showed that the query returned:

```text
3 columns
```
