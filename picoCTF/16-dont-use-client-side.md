# dont-use-client-side

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A password verification page where the entire verification logic — including the flag itself — was embedded in client-side JavaScript.

---

## Approach

### Step 1 — View Source
Pressed `Ctrl+U` — found a `verify()` function in JavaScript that checked the password by comparing substrings.

### Step 2 — Reassemble the Flag
The flag was split into 8 chunks of 4 characters each, checked in scrambled order:

```javascript
split = 4;
// Chunks checked out of order:
checkpass.substring(0, split)      == 'pico'  // pos 0
checkpass.substring(split, split*2) == 'CTF{' // pos 1
checkpass.substring(split*2, split*3) == 'no_c' // pos 2
checkpass.substring(split*3, split*4) == 'lien' // pos 3
checkpass.substring(split*4, split*5) == 'ts_p' // pos 4
checkpass.substring(split*5, split*6) == 'lz_2' // pos 5
checkpass.substring(split*6, split*7) == 'eb02' // pos 6
checkpass.substring(split*7, split*8) == 'b45}' // pos 7
```

Reassembled in order: `pico` + `CTF{` + `no_c` + `lien` + `ts_p` + `lz_2` + `eb02` + `b45}`

---

## Flag

```
picoCTF{no_clients_plz_2eb02b45}
```

---

## Lesson Learned

> Never put secrets, passwords, or flags in client-side JavaScript. Scrambling or splitting doesn't help — anyone can read the source and reassemble it. All authentication and verification must happen server-side.
