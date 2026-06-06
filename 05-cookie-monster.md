# Cookie Monster

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A login page with a hidden flag stored directly inside a cookie value — encoded in Base64 and sitting in plain sight.

---

## Approach

### Step 1 — Initial Recon
Landed on the login page and immediately checked F12 → Application → Cookies. Nothing suspicious yet.

### Step 2 — Triggering the Cookie
Entered a random username and password and submitted the form. Got redirected to another page.

Checked F12 → Application → Cookies again — a new cookie appeared:

```
Name:  secret_recipe
Value: cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzX0M0MzBBRTIwfQ%3D%3D
```

### Step 3 — Decoding the Value
The value ending in `%3D%3D` was URL encoded — `%3D` decodes to `=`.

So the actual value was:
```
cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzX0M0MzBBRTIwfQ==
```

Strings ending in `==` are a classic sign of **Base64 encoding**. Ran it through a Base64 decoder and got the flag immediately.

---

## Flag

```
picoCTF{c00k1e_m0nster_l0ves_c00kies_C430AE20}
```

---

## Root Cause

The developer stored sensitive data directly in a cookie value. Even though it was Base64 encoded — **Base64 is not encryption.** It's just encoding, trivially reversible by anyone with an internet connection.

This maps to **OWASP A02 — Cryptographic Failures**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Never store sensitive data in cookies | Cookies are visible to anyone with F12 |
| Use session IDs instead | Store sensitive data server-side, reference it with an ID |
| Never confuse encoding with encryption | Base64, ROT13, hex — none of these are encryption |

---

## Lesson Learned

> Cookies are CLIENT-SIDE storage — visible to every user. Never store passwords, flags, secrets, or any sensitive data in cookies. Store only session identifiers that reference server-side data.

> **Base64 is not encryption. It's just encoding.**
