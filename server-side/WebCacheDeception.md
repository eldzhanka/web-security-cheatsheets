# 🗄️ Web Cache Deception Cheatsheet

Complete methodology, bypass techniques, and exploitation vectors for Web Cache Deception labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Methodology

* Key Concept: Tricks the cache server into caching sensitive dynamic responses (e.g., `/my-account`) by appending static file extensions or path delimiters that the origin server strips/ignores.
* Target Victims: Authenticated users clicking a malicious link (`<script>` or `<img>` src pointing to payload path).

---

## 🛠️ Detailed Breakdown & Payloads

* **Exploiting path mapping for web cache deception:**
  - Leverage discrepancies in RESTful path handling where origin server maps `/my-account/wcd.js` back to `/my-account`, but cache rules match `.js` static extension:
  - Payload Vector: `https://target.com/my-account/wcd.js`

* **Exploiting path delimiters for web cache deception:**
  - Inject character delimiters (`/`, `;`, `?`, `%00`) to separate the dynamic endpoint from the static extension required by cache rules:
  - Delimiter Vectors: 
    - `https://target.com/my-account;wcd.js`
    - `https://target.com/my-account?wcd.js`
    - `https://target.com/my-account%00wcd.js`

* **Exploiting origin server normalization for web cache deception:**
  - Exploit scenarios where the origin server normalizes URL encoded dot segments (e.g., `%2f..%2f`) while the cache proxy evaluates raw path:
  - Payload Vector: `https://target.com/my-account%2f..%2fstatic/wcd.js`
  - Origin normalizes to `/my-account`, Cache sees `/static/wcd.js` extension.

* **Exploiting cache server normalization for web cache deception:**
  - Exploit scenarios where the cache server normalizes encoded dot segments/delimiters prior to matching cache rules:
  - Payload Vector: `https://target.com/my-account;%2f%2e%2e%2fstatic/wcd.js` or `https://target.com/profile;%2f%2e%2e%2f%2e%2e%2fstatic/wcd.js`

* **Exploiting exact-match cache rules for web cache deception:**
  - Exploit cache rules configured for exact static filenames or paths (e.g., `/favicon.ico` or `/robots.txt`) by abusing origin path normalization/routing:
  - Payload Vector: `https://target.com/my-account/..%2ffavicon.ico` or `https://target.com/my-account/..%2frobots.txt`

---
