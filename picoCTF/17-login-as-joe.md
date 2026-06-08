# Login as Joe (Logon)

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A login page where the goal is to log in as Joe and find the flag. The authentication was controlled entirely by client-side cookies — including an `admin` flag that could be manually toggled.

---

## Approach

### Step 1 — Register and Login
Registered with a random username and password and logged in successfully.

### Step 2 — Inspect Cookies
Opened Cookie Editor extension and found three suspicious cookies:

| Cookie Name | Value |
|-------------|-------|
| `name` | `admin` |
| `password` | `123` |
| `admin` | `false` |

The credentials were stored in plain text in cookies — and there was an `admin` flag set to `false`.

### Step 3 — Login with Real Credentials
Logged out and logged back in using `admin` / `123` — the credentials found in the cookies.

### Step 4 — Cookie Manipulation
Using Cookie Editor, changed the `admin` cookie value from `false` to `true` and refreshed the page.

Flag appeared immediately! 🎯

---

## Flag

```
picoCTF{th3_c0nsp1r4cy_l1v3s_4d184b0d}
```

---

## Root Cause

Two critical vulnerabilities combined:
1. **Credentials stored in cookies** — username and password visible in plain text client-side
2. **Authorization controlled by client-side cookie** — `admin=false` can be changed to `admin=true` by anyone

This maps to **OWASP A01 — Broken Access Control** and **OWASP A07 — Identification and Authentication Failures**.

---

## Lesson Learned

> Never store credentials or authorization state in client-side cookies. A cookie saying `admin=false` is not security — any user can change it to `admin=true`. Authorization must always be verified server-side.
