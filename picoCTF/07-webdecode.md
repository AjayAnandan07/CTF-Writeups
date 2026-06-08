# WebDecode

**Category:** Web Exploitation  
**Points:** 50  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A website with multiple pages (Home, About, Contact) where sensitive data was hidden inside the HTML source code — encoded in Base64 and tucked inside an HTML attribute.

---

## Approach

### Step 1 — Web Inspector
Challenge hint said "Do you know how to use the web inspector?" — opened F12 immediately after landing on the page. Nothing useful in cookies or network traffic.

### Step 2 — Viewing Page Source
Switched to `Ctrl+U` to view the raw HTML source code and navigated through each page.

In `about.html` found a suspicious HTML attribute:

```html
<section class="about" notify_true="cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfMDJjZGNiNTl9">
```

The `notify_true` attribute had no legitimate purpose — and its value looked like Base64.

### Step 3 — Decoding
Recognized the Base64 pattern (random alphanumeric string) and decoded it:

```
cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfMDJjZGNiNTl9
↓
picoCTF{web_succ3ssfully_d3c0ded_02cdcb59}
```

Flag retrieved! 🎯

---

## Flag

```
picoCTF{web_succ3ssfully_d3c0ded_02cdcb59}
```

---

## Root Cause

Sensitive data was stored directly in the HTML source code — visible to anyone who presses `Ctrl+U`. Base64 encoding provides zero security — it's trivially reversible.

This maps to **OWASP A02 — Cryptographic Failures**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Never store credentials or secrets in source code | Source code is public |
| Never use custom HTML attributes to hide data | Easily visible in inspector |
| Base64 is not encryption | Anyone can decode it instantly |
| Store secrets server-side only | Client side = attacker side |

---

## Encoding Recognition Cheatsheet
| Pattern | Encoding |
|---------|----------|
| Ends with `=` or `==` | Base64 |
| Only `a-f` and `0-9` | Hex |
| Shifted letters | ROT13 |
| `%3D`, `%2F` | URL Encoding |

---

## Lesson Learned

> The client side is the attacker's side. Anything in HTML, JavaScript, or CSS is visible to everyone. Never hide secrets in source code — encoded or not.
