# SQL Injection — Lab: SQL Injection Vulnerability Allowing Login Bypass

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-subverting-application-logic/sql-injection/lab-login-bypass)

## Objective

This lab contains a SQL injection vulnerability in the application's login function.

The objective is to perform a SQL injection attack that bypasses the authentication mechanism and logs in to the application as the `administrator` user.

---

## Initial Reconnaissance

I browsed the application and observed that it contained a login form.

When invalid credentials were submitted, the application returned an error indicating that the username or password was incorrect.

The login request was submitted using the HTTP `POST` method.

---

## Interesting Parameter

The login request uses `POST`, so the username and password parameters are sent in the request body rather than in the URL.

The interesting parameters were:

```text
username=<username>
password=<password>
```

The `username` parameter was selected for SQL injection testing.

---

## Testing

I tested a Boolean SQL injection condition in the `username` field while providing a random password in the `password` field.

The username input was:

```text
' OR 1=1--
```

The password field contained:

```text
password
```

The modified request caused the application to authenticate me as the `administrator` user.

---

## Observation

After submitting the modified input, I was successfully logged in as:

```text
administrator
```

I did not know the administrator's actual password.

This demonstrated that the authentication logic could be manipulated through the SQL injection vulnerability.

---

## Vulnerability

The application's login functionality is vulnerable to **SQL Injection**.

The `username` parameter can be manipulated so that user-controlled input changes the logic of the SQL query used for authentication.

---

## Exploitation

I exploited the SQL injection vulnerability in the username parameter to bypass the application's authentication check.

The successful injection caused the application to authenticate me as the administrator account without requiring the legitimate administrator password.

---

## Why It Worked

The vulnerability occurs because user-controlled input is incorporated into the SQL query without being safely parameterized.

Conceptually, an unsafe authentication query might look like:

```sql
SELECT * FROM users
WHERE username = '<username>'
AND password = '<password>';
```

The injected SQL changes the logic of the `WHERE` clause, causing the authentication condition to evaluate differently from what the developer intended.

The root cause is **improper handling of user input when constructing the SQL query**.

---

## Impact

An attacker could potentially bypass the application's authentication mechanism and gain access to an account without knowing the legitimate credentials.

In this lab, the demonstrated impact was:

* Authentication bypass.
* Unauthorized access to the `administrator` account.
* Access to functionality available to the authenticated administrator.

The exact impact in a real application would depend on the privileges and functionality associated with the compromised account.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements** instead of constructing SQL queries by concatenating user input.

Developers should also:

* Use prepared statements for database queries.
* Validate input appropriately.
* Apply least-privilege permissions to database accounts.
* Avoid exposing detailed database errors.
* Implement appropriate authentication and authorization controls.
* Perform security testing against authentication-related parameters.

Input filtering alone should not be relied upon as the primary defense against SQL injection.

---

## Tools

* Burp Suite
* Web Browser

---

## Lessons Learned

* How SQL injection can affect authentication mechanisms.
* How login requests work using the HTTP `POST` method.
* How user-controlled input can alter SQL query logic.
* How Boolean SQL injection can be used to test authentication logic.
* Why parameterized queries are important for preventing SQL injection.
* How SQL queries can influence application authentication.
* How to document the root cause, impact, and remediation of a vulnerability.

---

## Key Takeaway

This lab demonstrated that SQL injection can affect more than database searches or data retrieval.

When user input is directly incorporated into an authentication query, an attacker may be able to manipulate the query's logic and bypass the intended authentication mechanism.

The correct defense is to separate **SQL code from user-controlled data**, primarily through parameterized queries and prepared statements.
