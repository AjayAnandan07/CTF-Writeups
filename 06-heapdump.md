# Heapdump

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A web application where an endpoint produces the server memory dump which contains a hidden flag — exposed publicly through the API documentation.

---

## Approach

### Step 1 — Reading the API Documentation
The challenge description specifically mentioned an "API Documentation" article on the blog. Went straight to it.

The documentation listed every available API endpoint including one suspicious one:

```
GET /heapdump — Diagnosing the memory allocation
```

A memory dump endpoint — publicly accessible with no authentication.

### Step 2 — Downloading the Heap Dump
Visited the endpoint directly:

```
http://[target]/heapdump
```

The server responded with a downloadable file:

```
heapdump-1780748535203.heapsnapshot
```

An 11MB file containing the entire server memory at that point in time.

### Step 3 — Extracting the Flag
Used `grep` to search for the picoCTF flag format inside the massive dump file:

```bash
grep -o "picoCTF{[^}]*}" heapdump-*.heapsnapshot
```

Flag found immediately! 🎯

---

## Flag

```
picoCTF{Pat!3nt_15_Th3_K3y_a485f162}
```

---

## Root Cause

A diagnostic endpoint `/heapdump` was left publicly accessible in production with no authentication. Server memory dumps can contain extremely sensitive data including API keys, session tokens, database credentials, and user data.

This maps to **OWASP A05 — Security Misconfiguration**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Never store dump files in web-accessible directories | Anyone can download them |
| Disable public access to diagnostic endpoints | `/heapdump`, `/debug`, `/metrics` should require auth |
| Remove dumps after troubleshooting | No reason to keep them in production |
| Restrict to internal network only | Diagnostic tools are for developers, not users |
| Monitor for accidental exposure | Regular security assessments catch these |

---

## Lesson Learned

> Debug and diagnostic endpoints are for developers — never for the public. Always remove or protect them before pushing to production. A publicly accessible memory dump is essentially handing an attacker a snapshot of everything your server knows.

**Read the docs. The answer is often right there. 📖**
