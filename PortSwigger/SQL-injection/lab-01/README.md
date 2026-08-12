# SQL Injection Lab: SQL Injection Vulnerability in WHERE Clause Allowing Retrieval of Hidden Data

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-retrieving-hidden-data/sql-injection/lab-retrieve-hidden-data#)

## Objective

This lab contains a SQL injection vulnerability in the product category filter.

The application uses a SQL query similar to:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1
```

The objective is to exploit the SQL injection vulnerability so that the application displays one or more **unreleased products**.

---

## Initial Reconnaissance

I first browsed the application and observed that products could be filtered by category.

The category value was reflected in the URL.

---

## Interesting Parameter

The interesting parameter was:

```text
/filter?category=<ProductCategory>
```

The `category` parameter appeared to be directly involved in the application's database query, making it a candidate for SQL injection testing.

---

## Testing

I changed the value of the `category` parameter and tested a Boolean SQL expression:

```text
'+1=1--
```

The resulting request looked similar to:

```text
/filter?category='+OR+1=1--
```

The application's response changed significantly compared with the normal request.

This indicated that the input was being interpreted as part of the SQL statement rather than being treated only as ordinary category data.

### SQL Logic

The application's original query was conceptually:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1
```

By manipulating the `category` parameter, I was able to alter the logic of the `WHERE` clause.

The injected Boolean condition evaluated to true, and the remainder of the original query was commented out.

---

## Observation

After sending the modified request, the application displayed products that were not normally visible.

In particular, **unreleased products were displayed**, confirming that the SQL injection successfully altered the application's filtering logic.

---

## Vulnerability

**SQL Injection — WHERE Clause**

The `category` parameter is vulnerable because user-controlled input is incorporated into the SQL query without being safely parameterized.

The vulnerability allows an attacker to modify the logic of the database query.

---

## Exploitation

I exploited the vulnerable `category` parameter using a Boolean SQL injection condition.

The successful injection caused the application's original filtering condition to be bypassed, resulting in the display of unreleased products.

### Result

```text
Normal request
     ↓
Only released products
     ↓
SQL injection
     ↓
WHERE clause logic altered
     ↓
Unreleased products displayed
```

---

## Why It Worked

The application appears to construct the SQL query using user-controlled input.

Instead of treating the category value strictly as **data**, the database interprets portions of the supplied input as **SQL syntax**.

Because the query is not safely parameterized, the attacker can alter the logical conditions of the `WHERE` clause.

The important lesson is that simply filtering or encoding a few special characters is **not a reliable defense against SQL injection**.

---

## Impact

In this lab, the demonstrated impact was:

* Bypassing the intended product filtering logic.
* Retrieving products that should normally remain hidden.
* Accessing application data outside the intended authorization/filtering logic.

Depending on the SQL injection context, database privileges, and application design, SQL injection can potentially lead to more serious consequences such as unauthorized data disclosure or modification. However, those consequences were **not directly demonstrated in this lab**.

---

## Remediation

The primary defense is to use **parameterized queries / prepared statements** rather than constructing SQL queries by concatenating user input.

Conceptually:

```text
User Input
    ↓
Parameterized Query
    ↓
Database
```

The application should also:

* Use prepared statements.
* Apply appropriate input validation.
* Use the principle of least privilege for database accounts.
* Avoid exposing detailed database errors to users.
* Perform security testing on database-facing parameters.

**Important:** Input filtering alone should not be relied upon as the primary SQL injection defense.

---

## Tools

* Google Chrome
* Burp Suite / Browser Developer Tools (if applicable)

---

## Lessons Learned

* What SQL injection is.
* How SQL injection can alter a `WHERE` clause.
* How Boolean conditions can be used to test SQL injection.
* How user-controlled parameters can influence database queries.
* How to identify potentially vulnerable parameters.
* Why parameterized queries are preferred over input filtering.
* How to verify a vulnerability by comparing application responses.

---

## Key Takeaway

The main lesson from this lab was that SQL injection is not simply about finding a particular payload.

The important process is:

```text
Identify input
      ↓
Understand its SQL context
      ↓
Test application behavior
      ↓
Determine whether SQL logic can be altered
      ↓
Demonstrate the security impact
      ↓
Understand the root cause
      ↓
Apply the correct remediation
```

This lab demonstrated how an improperly handled category parameter could be manipulated to bypass the `released` condition and expose hidden products.
