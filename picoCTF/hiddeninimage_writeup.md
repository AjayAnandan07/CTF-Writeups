# PicoCTF – Hidden in Image
**Category:** Forensics  
**Difficulty:** Easy  
**Flag:** `picoCTF{h1dd3n_1n_1m4g3_54e31417}`

---

## Description

> You're given a seemingly ordinary JPG image. Something is tucked away out of sight inside the file. Your task is to discover the hidden payload and extract the flag.

---

## Exploitation

### Step 1 – Inspect the JPEG

```bash
file img.jpg
strings img.jpg | grep -i "pico"
```

The `file` output revealed a JPEG comment field:

```
comment: "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9"
```

### Step 2 – Decode the Comment

```bash
echo "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9" | base64 -d
# steghide:cEF6endvcmQ=
```

Two pieces of info:
- Tool to use: `steghide`
- Password (base64): `cEF6endvcmQ=`

```bash
echo "cEF6endvcmQ=" | base64 -d
# pAzzword
```

### Step 3 – Extract Hidden Data

```bash
steghide extract -sf img.jpg -p "pAzzword"
# wrote extracted data to "flag.txt"
cat flag.txt
```

---

## Flag

```
picoCTF{h1dd3n_1n_1m4g3_54e31417}
```

---

## Key Takeaways

- JPEG files have a comment field that can hide hints or data
- `strings` is a quick way to spot readable data in binary files
- `steghide` is a common steganography tool for hiding data inside JPEG/BMP files
- Always check: file metadata, strings, EXIF data, and steganography tools on image forensics challenges
- Common forensics toolkit: `strings`, `file`, `exiftool`, `binwalk`, `steghide`, `pdfinfo`

{forensics}
