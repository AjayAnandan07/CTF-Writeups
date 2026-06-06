# Dev Backdoor

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A restricted web portal where a person of interest is believed to be hiding sensitive data. The email `ctf-player@picoctf.org` is known but the password isn't — however something feels off, like the developer left a secret way in.

---

## Approach

### Step 1 — Source Code Recon
As soon as I landed on the login page, knowing the challenge hinted at a developer mistake, I immediately pressed `Ctrl+U` to view the page source code.

No messing around with the login form first — straight to the source.

### Step 2 — Finding the Hidden Comment
While reading through the HTML, came across a suspicious commented-out line encoded in **ROT13**:

```
<!-- ABGR: Wnpx - grzcbenel olcnff: hfr urnqre "K-Qri-Npprff: lrf" -->
<!-- Remove before pushing to production! -->
```

The second line gave it away immediately — *"Remove before pushing to production!"* — classic developer mistake.

### Step 3 — Decoding ROT13
Copied the encoded message and ran it through an online ROT13 decoder. The decoded message revealed:

```
NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes"
```

A hardcoded backdoor left in the source code by the developer — hidden using ROT13 encoding, which is not encryption at all.

### Step 4 — Exploiting the Backdoor
Using a **Header Editor** browser extension:
1. Added the header `X-Dev-Access` with value `yes`
2. Used the known email `ctf-player@picoctf.org` with any random password
3. Logged in successfully — the header bypassed authentication entirely
4. Found the flag inside the portal

---

## Flag

```
picoCTF{brut4_f0rc4_cbb8faa7}
```

---

## Root Cause

A careless developer mistake — debug/backdoor code was left in the source code and pushed to production without a proper review. The ROT13 "encoding" provided zero real protection since it's trivially reversible.

This maps to **OWASP A05 — Security Misconfiguration**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Code review before every production push | Catch leftover debug code |
| Remove ALL commented-out backdoors | Comments are visible to anyone |
| Never use ROT13 as "security" | It's not encryption — it's just shifting letters |
| Implement proper authentication | No header should ever bypass login |

---

## Lesson Learned

> The source code of a webpage is PUBLIC. Never leave sensitive information, backdoors, or debug notes in HTML comments — encoded or not. Always review code before pushing to production.

---

## Key Takeaway

**ROT13 is not encryption.** It's a Caesar cipher that shifts letters by 13 positions. Any online tool decodes it in seconds. Never mistake obscurity for security.
