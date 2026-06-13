# picoCTF — Logs Analysis Writeup

## Challenge Info

| Field    | Details         |
|----------|-----------------|
| Category | Forensics       |
| File     | `logs.txt`      |

---

## Initial Observation

Opening `logs.txt` reveals a single enormous block of encoded text instead of typical log entries. The first characters — `iVBORw0KGgo` — are an immediate red flag.

> **Key insight:** `iVBORw0KGgo` is the Base64 encoding of the PNG magic bytes `\x89PNG\r\n\x1a\n`. Any time you see this prefix in a "text" file, you're looking at a hidden image.

---

## Step 1 — Decode the Base64

```bash
base64 -d logs.txt > decoded_output
file decoded_output
# decoded_output: PNG image data, 896 x 1152, 8-bit/color RGB, non-interlaced
```

The log file was entirely a Base64-encoded PNG image.

---

## Step 2 — Inspect the Image

Check PNG chunks for any hidden metadata (`tEXt`, `zTXt`, ancillary chunks):

```python
import struct

with open('decoded_output', 'rb') as f:
    data = f.read()

pos = 8  # skip PNG signature
while pos < len(data):
    length = struct.unpack('>I', data[pos:pos+4])[0]
    chunk_type = data[pos+4:pos+8].decode('ascii', errors='replace')
    print(f'Chunk: {chunk_type}, Length: {length}')
    pos += 12 + length
```

**Result:** Only standard `IHDR`, `IDAT`, and `IEND` chunks — nothing hidden in metadata. The data must be **visible in the image itself**.

---

## Step 3 — OCR the Image

```bash
apt install tesseract-ocr

python3 -c "
import pytesseract
from PIL import Image
img = Image.open('decoded_output')
print(pytesseract.image_to_string(img))
"
```

Among the OCR output:

```
7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F62653836303237397D
```

A hex string rendered visually inside the image.

---

## Step 4 — Decode the Hex

```python
python3 -c "
print(bytes.fromhex('7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F62653836303237397D').decode())
"
```

---

## Flag

```
picoCTF{forensics_analysis_is_amazing_be860279}
```

---

## The Full Chain

```
logs.txt
   │
   │  Base64 decode
   ▼
PNG image (896x1152)
   │
   │  OCR (Tesseract)
   ▼
Hex string: 7069636F...7D
   │
   │  hex decode
   ▼
picoCTF{forensics_analysis_is_amazing_be860279}
```

---

## Lessons Learned

- **`iVBORw0KGgo` = Base64 PNG** — memorise this pattern; it appears constantly in CTFs.
- **`file` command first** — always run `file` on decoded output before assuming its content type.
- **When metadata is clean, look at pixels** — not all hidden data lives in PNG chunks; sometimes it's just rendered text in the image.
- **OCR is a valid forensics tool** — especially for challenges where text is embedded visually inside an image.
