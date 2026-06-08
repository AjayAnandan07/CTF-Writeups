# GET aHead

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A Red/Blue choice website where clicking Red sends a GET request and clicking Blue sends a POST request. The flag was hidden in the HTTP response headers — only visible when using the HEAD method.

---

## Approach

### Step 1 — Reading the Challenge Name
"GET aHead" — the name itself is the hint:
- **GET** — HTTP method
- **aHead** — HEAD is another HTTP method
- Flag is in the **headers**

### Step 2 — Trying HEAD Method
Used curl with the `--head` flag to send a HEAD request:

```bash
curl -v --head http://[target]/index.php
```

### Step 3 — Flag in Response Headers
The response contained a custom header:

```
flag: picoCTF{r3j3ct_th3_du4l1ty_8b13f07}
```

The flag was sitting in the HTTP response headers the entire time — invisible to regular browser traffic but exposed via HEAD request.

---

## Flag

```
picoCTF{r3j3ct_th3_du4l1ty_8b13f07}
```

---

## HTTP Methods Learned

| Method | Purpose |
|--------|---------|
| GET | Retrieve a resource |
| POST | Send data to server |
| HEAD | Same as GET but returns ONLY headers, no body |
| PUT | Update a resource |
| DELETE | Delete a resource |

---

## Lesson Learned

> Never store sensitive data in HTTP response headers — they're visible to anyone using curl, Burp Suite, or browser dev tools. The HEAD method is a real pentester tool for checking what headers a server exposes without downloading the full response body.
