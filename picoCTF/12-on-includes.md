# On Includes

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A webpage where the flag was split across two external files — half in a CSS comment and half in a JavaScript comment.

---

## Approach

### Step 1 — View Source
Pressed `Ctrl+U` — found two external files linked: `style.css` and `script.js`.

### Step 2 — Checking CSS
Opened `style.css` — found first half of flag in a comment:
```css
/* picoCTF{1nclu51v17y_1of2_ */
```

### Step 3 — Checking JS
Opened `script.js` — found second half:
```javascript
// f7w_2of2_6edef411}
```

### Step 4 — Combining
Joined both halves together to get the complete flag.

---

## Flag

```
picoCTF{1nclu51v17y_1of2_f7w_2of2_6edef411}
```

---

## Lesson Learned

> Always check ALL linked files — CSS, JS, images, fonts. Secrets can be split across multiple files but `Ctrl+U` + following every link exposes them all.

> Comments in CSS and JS are just as visible as HTML comments. Never store secrets in any client-side file.
