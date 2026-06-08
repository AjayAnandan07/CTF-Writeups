# Intro to Burp

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Status: 🔄 In Progress

This challenge involves brute forcing a 2FA OTP using Burp Suite Intruder.

Coming back to this one after getting more comfortable with Burp Suite Community Edition limitations.

---

## What I Know So Far

- Website has registration + 2FA OTP page
- OTP submitted via POST request to `/dashboard`
- Session cookie and CSRF token required
- Burp Intruder rate limited in Community Edition
- Plan: Write a Python brute force script instead

---

## TODO
- [ ] Write Python brute force script using `requests`
- [ ] Handle CSRF token in each request
- [ ] Identify OTP length
- [ ] Capture the flag
