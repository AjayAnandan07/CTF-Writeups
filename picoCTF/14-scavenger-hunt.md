# Scavenger Hunt

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A simple website built with HTML, CSS, and JS where the flag was split across 5 different files — each hidden in a comment or special server file.

---

## Approach

### Step 1 — HTML Source
`Ctrl+U` revealed Part 1 in an HTML comment:
```
<!-- Here's the first part of the flag: picoCTF{t -->
```

### Step 2 — CSS File
Opened `mycss.css` — found Part 2 in a comment:
```
/* Here's part 2: h4ts_4_l0 */
```

### Step 3 — JS File
Opened `myjs.js` — no flag but a hint:
```
/* How can I keep Google from indexing my website? */
```
→ Pointed to `robots.txt`

### Step 4 — robots.txt
Visited `/robots.txt` — found Part 3 and another hint:
```
# Part 3: t_0f_pl4c
# I think this is an apache server... can you Access the next flag?
```
→ "Access" pointed to `.htaccess`

### Step 5 — .htaccess
Visited `/.htaccess` — found Part 4 and another hint:
```
# Part 4: 3s_2_lO0k
# I love making websites on my Mac, I can Store a lot of information there.
```
→ "Mac" + "Store" pointed to `.DS_Store`

### Step 6 — .DS_Store
Visited `/.DS_Store` — found Part 5:
```
Part 5: _9588550}
```

---

## Flag

```
picoCTF{th4ts_4_l0t_0f_pl4c3s_2_lO0k_9588550}
```

---

## Key Files Every Pentester Checks

| File | What it reveals |
|------|----------------|
| HTML/CSS/JS comments | Hidden data, credentials, hints |
| `/robots.txt` | Search engine rules — often exposes hidden paths |
| `/.htaccess` | Apache server config — often left exposed |
| `/.DS_Store` | Mac metadata accidentally uploaded to server |

---

## Lesson Learned

> Always check beyond just the main page. `robots.txt`, `.htaccess`, and `.DS_Store` are real files pentesters check on every engagement — they're often accidentally left public and reveal sensitive server information.
