# Old Session

**Category:** Web Exploitation  
**Points:** 100  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A website with the rule "Login once and stay logged in forever" — featuring bad session misconfiguration and an exposed `/sessions` page that leaks active session cookies.

---

## Approach

### Step 1 — Reconnaissance
Registered and logged in as a normal user to understand the application flow. After logging in, the homepage displayed a comment feed from multiple users.

### Step 2 — Finding the Vulnerability
While reading through the comments on the homepage, one user — `mary_jones_8992` — had left a comment saying:

> *"Hey I found a strange page at /sessions"*

This was an information disclosure through the application itself. Modified the URL by appending `/sessions` to the base URL.

### Step 3 — Exploiting the Session Leak
The `/sessions` page exposed all active session tokens in plain text, including:

```
session: NAcR0zstNIRnUNc51NekMeb1CsBdZW6w8AH7ojlah4E — {'_permanent': True, 'key': 'admin'}
session: VjEQF4wGe-yXPvfnIqoBYOUrcj2vKShTdCZZP6TufnQ — {'_permanent': True, 'key': '123'}
```

The Admin's session token was visible and marked as `_permanent: True` — meaning it never expires.

### Step 4 — Session Hijacking
Using the **Cookie Editor** browser extension:
1. Replaced my current session cookie with the Admin's session token
2. Refreshed the page
3. Was now authenticated as Admin
4. Found the flag on the homepage

---

## Flag

```
picoCTF{s3t_s3ss10n_3xp1rat10n5_7139c037}
```

---

## Root Cause

Two core vulnerabilities made this attack possible:

1. **No session expiration** — `_permanent: True` meant Admin's session was valid indefinitely since 2023
2. **Broken Access Control** — The `/sessions` page was accessible to any logged-in user with no role restriction, exposing all active session tokens in plain text

---

## How to Fix

| Fix | Implementation |
|-----|----------------|
| Proper session expiration | Set `PERMANENT_SESSION_LIFETIME` to a short duration (e.g. 30 minutes) |
| Role Based Access Control | Restrict `/sessions` to admin-only or remove it entirely from production |
| Never expose session tokens | Session data should never be readable by other users |

---

## Lesson Learned

> Always implement session expiration and Role Based Access Control. A page that leaks session tokens is essentially handing attackers the keys to every account on the platform.

This vulnerability maps to **OWASP A07 — Identification and Authentication Failures**.
