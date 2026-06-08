# Inspect HTML

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A webpage about Histiaeus with the flag hidden in an HTML comment — visible only through page source.

---

## Approach

Challenge title said "Inspect HTML" — pressed `Ctrl+U` immediately. Found the flag sitting in an HTML comment at the bottom of the page:

```html
<!--picoCTF{1n5p3t0r_0f_h7ml_8113f7e2}-->
```

Done. 😄

---

## Flag

```
picoCTF{1n5p3t0r_0f_h7ml_8113f7e2}
```

---

## Lesson Learned

> HTML comments are NOT hidden. They're visible to anyone who presses `Ctrl+U`. Never put secrets, credentials, or flags in HTML comments.
