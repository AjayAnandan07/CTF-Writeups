# Bookmarklet

**Category:** Web Exploitation  
**Points:** 50  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A flag distribution website with a bookmarklet containing encrypted flag data and its own decryption logic — all exposed in client-side JavaScript.

---

## Approach

### Step 1 — View Source
Pressed `Ctrl+U` to view page source. Found a JavaScript bookmarklet in a textarea containing:
- An encrypted flag string
- A decryption function using a XOR-style cipher
- The decryption key `"picoctf"` hardcoded in plain text

### Step 2 — Running the Code
Instead of saving it as a bookmark, copied the JavaScript directly into the browser console (F12 → Console) and ran it.

The decryption function executed and displayed the flag in an alert box.

---

## Flag

```
picoCTF{p@g3_turn3r_0c0d211f}
```

---

## Root Cause

The encrypted flag AND the decryption key were both stored in client-side JavaScript — visible to anyone who views the source. Having both the ciphertext and the key in the same place provides zero security.

This maps to **OWASP A02 — Cryptographic Failures**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Never store secrets in client-side code | JavaScript is public |
| Never store encryption keys alongside encrypted data | Defeats the purpose of encryption |
| Decrypt server-side only | Client never sees the key or raw flag |

---

## Lesson Learned

> If you encrypt something but include the key right next to it — you haven't secured anything. Real encryption keeps the key secret and separate. Always decrypt sensitive data server-side.
