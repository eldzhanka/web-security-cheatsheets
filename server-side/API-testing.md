# 🔌 API Testing Vulnerabilities Cheatsheet

Complete methodology, bypass techniques, and exploitation vectors for API Testing labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Methodology

* Target Common Endpoints: `/api/`, `/v1/`, `/v2/`, `/swagger.json`, `/openapi.json`
* Key Testing Areas:
  - Hidden / Unlinked API Documentation Discovery
  - HTTP Method Override (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)
  - Mass Assignment / Parameter Tampering in JSON Request Bodies

---

## 🛠️ Detailed Breakdown & Payloads

* **Exploiting an API endpoint using documentation:**
  - Discover exposed OpenAPI/Swagger documentation or API endpoint definitions by fuzzing paths or inspecting JS files:
  - Target Path: `/api/swagger.v1.json` or `/api/docs`
  - Action: Locate administrative user endpoints such as `DELETE /api/users/carlos` and execute directly via API.

* **Finding and exploiting an unused API endpoint:**
  - Test alternate HTTP methods on known resources to discover hidden actions (e.g., changing `GET` or `POST` to `PATCH` or `DELETE`):
  - Example: Intercept item modification request, change method to `PATCH /api/products/1/price` with payload `{"price": 0}`.

* **Exploiting a mass assignment vulnerability:**
  - Inspect GET response structures for internal object properties and include sensitive attributes in `POST`/`PATCH` JSON requests:
  - Request Payload:
    `POST /api/checkout`
    `{"chosen_discount": {"percentage": 100}}`
