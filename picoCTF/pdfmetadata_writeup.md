# PicoCTF – Hidden Confidential Document
**Category:** Forensics  
**Difficulty:** Easy  
**Points:** Unknown  
**Flag:** `picoCTF{puzzl3d_m3tadata_f0und!_0e2de5a1}`

---

## Description

> You've stumbled upon a peculiar PDF filled with what seems like nothing more than garbled nonsense. Amidst the chaos lies a hidden treasure—an elusive flag waiting to be uncovered. Uncover the flag within the metadata.

---

## Analysis

PDF files contain metadata fields like Author, Producer, Title, etc. These fields can be used to hide data. The challenge hints at metadata — so the first step is extracting it.

---

## Exploitation

### Step 1 – Extract PDF Metadata

```bash
pdfinfo confidential.pdf
```

Output:
```
Author: cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV8wZTJkZTVhMX0=
Producer: PyPDF2
Pages: 1
...
```

The `Author` field contains a suspicious base64 string.

### Step 2 – Decode Base64

```bash
echo "cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV8wZTJkZTVhMX0=" | base64 -d
```

Output:
```
picoCTF{puzzl3d_m3tadata_f0und!_0e2de5a1}
```

---

## Flag

```
picoCTF{puzzl3d_m3tadata_f0und!_0e2de5a1}
```

---

## Key Takeaways

- PDF metadata fields (Author, Title, Subject, Keywords) are common hiding spots in forensics challenges
- Always run `pdfinfo` or `exiftool` on any suspicious file
- Base64 strings are recognizable by their character set and `=` padding at the end
- `exiftool` is another powerful tool for extracting metadata from any file type

{forensics}
