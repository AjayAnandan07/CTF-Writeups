# Robots

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A website where the flag was hidden in a page referenced by `robots.txt` — a file meant to tell search engines what NOT to index, but ironically revealing hidden paths to anyone who reads it.

---

## Approach

### Step 1 — Check robots.txt
Visited `/robots.txt` directly — found a disallowed path:

```
Disallow: /cc6b1.html
```

### Step 2 — Visit the Hidden Page
Navigated to `http://[target]/cc6b1.html` — flag was there!

---

## Flag

```
picoCTF{ca1cu1at1ng_Mach1n3s_cc6b1}
```

---

## Lesson Learned

> `robots.txt` is public! Telling search engines NOT to index a page doesn't hide it — it actually advertises it to anyone who checks `robots.txt`. Never rely on obscurity for security. If a page should be private, protect it with authentication.
