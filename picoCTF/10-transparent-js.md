# Transparent JS (Secure Customer Portal)

**Category:** Web Exploitation  
**Points:** —  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A "secure" customer portal where the entire authentication logic — including admin credentials — was stored in a public JavaScript file.

---

## Approach

### Step 1 — View Source
Landed on a login page. Pressed `Ctrl+U` — noticed the form posted to `login.php`.

### Step 2 — Following the Trail
Visited `login.php` source — found it loaded a suspicious file called `secure.js` and contained a hidden admin form posting to `admin.php`.

### Step 3 — Reading secure.js
Opened `secure.js` directly in the browser. Found the entire authentication function:

```javascript
function checkPassword(username, password) {
  if( username === 'admin' && password === 'strongPassword098765' ) {
    return true;
  } else {
    return false;
  }
}
```

Credentials sitting in plain text in a public JavaScript file. 😂

### Step 4 — Logging In
Used `admin` / `strongPassword098765` to log in and got the flag.

---

## Flag

```
picoCTF{j5_15_7r4n5p4r3n7_05df90c8}
```

---

## Root Cause

Authentication was handled entirely client-side in JavaScript — publicly accessible to anyone. The credentials were hardcoded in plain text in `secure.js`.

This maps to **OWASP A07 — Identification and Authentication Failures**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Never do authentication client-side | JavaScript is public |
| Never hardcode credentials anywhere | Especially not in public files |
| Always authenticate server-side | Client never sees the comparison logic |
| Hash passwords properly | Even server-side, never store plaintext |

---

## Lesson Learned

> "Secure" in the filename means nothing if the contents are public. Authentication must always happen server-side. Client-side JavaScript authentication is not authentication — it's just a suggestion. 😄
