# Web Security Cheatsheets

[![BSCP Certified](https://img.shields.io/badge/BSCP-Certified-orange?style=for-the-badge&logo=burpsuite)](https://portswigger.net/web-security/e/c/6310b95d46cb5a3b)

Personal web security notes, payloads, and bypasses compiled during preparation for the **Burp Suite Certified Practitioner (BSCP)** certification.

Having these notes saved me a ton of time during the exam. You can also use and tweak these payloads for your own exploits.

> **Note:** I didn't include topics like [Business Logic Vulnerabilities](https://portswigger.net/web-security/logic-flaws) or [Race Conditions](https://portswigger.net/web-security/race-conditions) here. I recommend checking out the [PortSwigger Learning Paths](https://portswigger.net/web-security/learning-paths) for those.

---

## 📂 Table of Contents

### 🛡️ Server-Side Vulnerabilities
* [Authentication Vulnerability](server-side/Authentication-Vulnerability.md)
* [OS Command Injection](server-side/OS-Command-Injection.md)
* [Path traversal](server-side/Path-traversal-payloads.md)
* [API Testing](server-side/API-testing.md)
* [File Upload bypass](server-side/File-Upload-bypass.md)
* [SQL Injection (SQLi)](server-side/SQL-injection.md)
* [NoSQL Injection](server-side/NoSQLi.md)
* [Server Side Request Forgery](server-side/SSRF.md)
* [Web Cache Deception](server-side/WebCacheDeception.md)
* [XXE](server-side/XXE.md)

### 🛡️ Client-Side Vulnerabilities
* [CORS](client-side/CORS.md)
* [CSRF](client-side/CSRF.md)
* [DOM based vulnerabilities](client-side/DOM-based-vulnerabilities.md)
* [WebSocket vulnerabilities](client-side/WebSocket-Vulnerabilities.md)
* [XSS](client-side/XSS.md)

### 🛡️ Advanced Topics
* [GraphQL API](advanced-topics/GraphQL-API.md)
* [Insecure Deserialization](advanced-topics/Insecure-Deserialization.md)
* [Server Side Template Injection](advanced-topics/Server-Side-Template-Injection.md)
* [Web Cache Poisoning](advanced-topics/Web-Cache-Poisoning.md)
* [Web LLM Attacks](Web-LLM-Attacks.md)
