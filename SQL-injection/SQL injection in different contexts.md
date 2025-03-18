# SQL injection in different contexts

In the previous labs, you used the query string to inject your malicious SQL payload. However, you can perform SQL injection attacks using any controllable input that is processed as a SQL query by the application. For example, some websites take input in JSON or XML format and use this to query the database.

These different formats may provide different ways for you to obfuscate attacks that are otherwise blocked due to WAFs and other defense mechanisms. Weak implementations often look for common SQL injection keywords within the request, so you may be able to bypass these filters by encoding or escaping characters in the prohibited keywords. For example, the following XML-based SQL injection uses an XML escape sequence to encode the S character in SELECT:

```
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```

This will be decoded server-side before being passed to the SQL interpreter.

## Solving the Lab: SQL injection with filter bypass via XML encoding

### Identify the vulnerability

- Observe that the stock check feature sends the productId and storeId to the application in XML format.

- Send the POST /product/stock request to Burp Repeater.

- In Burp Repeater, probe the storeId to see whether your input is evaluated. For example, try replacing the ID with mathematical expressions that evaluate to other potential IDs, for example:

```
<storeId>1+1</storeId>
```

- Observe that your input appears to be evaluated by the application, returning the stock for different stores.

- Try determining the number of columns returned by the original query by appending a UNION SELECT statement to the original store ID:

```
<storeId>1 UNION SELECT NULL</storeId>
```

- Observe that your request has been blocked due to being flagged as a potential attack.

### Bypass the WAF (Web Application Firewall)

- As you're injecting into XML, try obfuscating your payload using XML entities. One way to do this is using the Hackvertor extension. Just highlight your input, right-click, then select Extensions > Hackvertor > Encode > dec_entities/hex_entities.

- Resend the request and notice that you now receive a normal response from the application. This suggests that you have successfully bypassed the WAF.

### Craft an exploit

- Pick up where you left off, and deduce that the query returns a single column. When you try to return more than one column, the application returns 0 units, implying an error.

- As you can only return one column, you need to concatenate the returned usernames and passwords, for example:

```
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>
```

