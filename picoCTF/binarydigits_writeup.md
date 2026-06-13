# PicoCTF – Binary Digits
**Category:** Forensics  
**Difficulty:** Easy  
**Points:** 100  
**Author:** Yahaya Meddy  
**Flag:** `picoCTF{h1dd3n_1n_th3_b1n4ry_67bd9b59}`

---

## Description

> This file doesn't look like much... just a bunch of 1s and 0s. But maybe it's not just random noise. Can you recover anything meaningful from this?

---

## Analysis

The file `digits.bin` contains a long string of `1`s and `0`s — 71,192 characters total, which is a multiple of 8. This strongly suggests it's binary-encoded data where every 8 bits represents one byte.

---

## Exploitation

### Step 1 – Decode Binary to Bytes

```python
with open("digits.bin", "r") as f:
    data = f.read().strip()

byte_array = bytearray()
for i in range(0, len(data) - len(data) % 8, 8):
    byte = data[i:i+8]
    byte_array.append(int(byte, 2))

with open("decoded.jpg", "wb") as f:
    f.write(byte_array)
```

### Step 2 – Identify File Type

The first decoded bytes were `ÿØÿà JFIF` — the magic header for a **JPEG image**.

### Step 3 – Open the Image

Opening `decoded.jpg` revealed a white image with the flag written in red text.

---

## Flag

```
picoCTF{h1dd3n_1n_th3_b1n4ry_67bd9b59}
```

---

## Key Takeaways

- Binary strings (1s and 0s) with length divisible by 8 are likely byte-encoded data
- Always check the first few decoded bytes for **magic headers** to identify file type
- Common magic headers: `FFD8FF` (JPEG), `89504E47` (PNG), `25504446` (PDF), `504B0304` (ZIP)
- Steganography/forensics often hides data in plain sight by encoding it differently

{forensics}
