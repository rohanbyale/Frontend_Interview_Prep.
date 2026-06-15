# 🏗️ HTML Interview Questions & Answers

---

## 1. What is semantic HTML? Why does it matter?

**🧠 Simple Explanation:**
Semantic HTML means using tags that **describe their meaning**, not just their appearance.

❌ Non-semantic:
```html
<div class="header">...</div>
<div class="nav">...</div>
```

✅ Semantic:
```html
<header>...</header>
<nav>...</nav>
<main>...</main>
<article>...</article>
<footer>...</footer>
```

**Why it matters:**
- Screen readers understand the page structure (Accessibility)
- Search engines rank it better (SEO)
- Other developers can read your code faster (Maintainability)

---

## 2. What's the difference between `<div>` and `<span>`?

| Feature | `<div>` | `<span>` |
|---------|---------|---------|
| Type | Block-level | Inline |
| Takes full width | ✅ Yes | ❌ No |
| Use case | Group sections | Highlight part of text |

```html
<!-- div takes full row -->
<div style="background:yellow">I am a block</div>

<!-- span only wraps the text -->
<p>Hello <span style="color:red">World</span></p>
```

---

## 3. What are `data-*` attributes?

They let you store **custom data** on HTML elements without using hidden inputs or JS variables.

```html
<button data-user-id="42" data-role="admin" onclick="handleClick(this)">
  Click Me
</button>

<script>
function handleClick(el) {
  console.log(el.dataset.userId); // "42"
  console.log(el.dataset.role);   // "admin"
}
</script>
```

**When to use:** Storing IDs, config, or state directly on DOM elements.

---

## 4. What is the `defer` vs `async` attribute on `<script>`?

Both load scripts **without blocking HTML parsing**, but differ in execution:

```html
<!-- Normal: blocks HTML parsing -->
<script src="app.js"></script>

<!-- async: downloads in parallel, runs IMMEDIATELY when ready (order not guaranteed) -->
<script async src="analytics.js"></script>

<!-- defer: downloads in parallel, runs AFTER HTML is fully parsed (order guaranteed) -->
<script defer src="app.js"></script>
```

| | `async` | `defer` |
|--|---------|---------|
| Download | Parallel | Parallel |
| Execution | As soon as downloaded | After HTML parsed |
| Order guaranteed | ❌ No | ✅ Yes |
| Best for | Independent scripts (analytics) | App scripts |

---

## 5. What is the difference between `<link>` and `@import` for CSS?

```html
<!-- <link> loads CSS in parallel with HTML — FASTER ✅ -->
<link rel="stylesheet" href="style.css">
```

```css
/* @import loads CSS sequentially — SLOWER ❌ */
@import url('style.css');
```

Always prefer `<link>` for performance. `@import` is fine inside CSS files for organizing code.

---

## 6. What is `<meta viewport>` and why is it important?

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

Without this, mobile browsers **zoom out** to show the desktop version.

With this tag:
- `width=device-width` → page width = phone screen width
- `initial-scale=1.0` → no zoom on load

This is the **first step** of making a site mobile-friendly.

---

## 7. What is `aria-*` and why should you use it?

ARIA (Accessible Rich Internet Applications) attributes help **screen readers** understand your UI.

```html
<!-- Without ARIA — screen reader says "button" -->
<button onclick="openModal()">✕</button>

<!-- With ARIA — screen reader says "Close dialog button" -->
<button onclick="openModal()" aria-label="Close dialog">✕</button>

<!-- For live regions (dynamic content) -->
<div aria-live="polite" id="status">Loading...</div>
```

Common ARIA attributes:
- `aria-label` — gives a name to an element
- `aria-hidden="true"` — hides from screen readers (decorative icons)
- `aria-expanded` — shows open/close state (menus, accordions)
- `aria-required` — marks required form fields
- `role` — defines what the element IS (e.g., `role="dialog"`)

---

## 8. What is the difference between `id` and `class`?

| | `id` | `class` |
|--|------|---------|
| Unique per page | ✅ Must be unique | ❌ Can repeat |
| CSS selector | `#myId` | `.myClass` |
| JS selector | `getElementById` | `getElementsByClassName` |
| Specificity | Higher | Lower |

```html
<p id="intro">This is unique on the page</p>
<p class="highlight">Many paragraphs can share this</p>
```

---

## 9. What is an `iframe` and when should you avoid it?

An `<iframe>` embeds another HTML page inside your page.

```html
<iframe src="https://example.com" width="600" height="400"></iframe>
```

**Avoid iframes when:**
- Security matters (clickjacking risk)
- SEO matters (search engines may not crawl iframe content)
- Performance matters (iframes block onload event)

**Use iframes for:** YouTube embeds, Google Maps, payment widgets (Stripe, PayPal).

---

## 10. What happens when you type a URL and press Enter? (HTML perspective)

1. Browser checks **DNS** → finds server IP
2. Browser makes **HTTP request** → server sends back HTML
3. Browser **parses HTML** top to bottom
4. When it finds `<link>` → fetches CSS
5. When it finds `<script>` → may pause parsing (unless defer/async)
6. When it finds `<img>` → fetches image
7. Browser builds **DOM tree** from HTML
8. Browser builds **CSSOM** from CSS
9. Combines both into **Render Tree**
10. **Paints** pixels on screen

---

## 11. What is the difference between `GET` and `POST` in forms?

```html
<!-- GET — data goes in URL (visible) -->
<form method="GET" action="/search">
  <input name="query" value="shoes">
  <!-- Submits to: /search?query=shoes -->
</form>

<!-- POST — data goes in request body (hidden) -->
<form method="POST" action="/login">
  <input name="password" type="password">
  <!-- Password NOT visible in URL -->
</form>
```

| | GET | POST |
|--|-----|------|
| Data location | URL query string | Request body |
| Visible in URL | ✅ Yes | ❌ No |
| Bookmarkable | ✅ Yes | ❌ No |
| Use for | Search, filters | Login, payments |
| Max data size | ~2000 chars | No practical limit |

---

## 12. What is `localStorage` vs `sessionStorage` vs `cookies`?

| | localStorage | sessionStorage | Cookies |
|--|-------------|----------------|---------|
| Expires | Never | When tab closes | Custom |
| Size | ~5MB | ~5MB | ~4KB |
| Sent to server | ❌ No | ❌ No | ✅ Yes |
| Accessible via JS | ✅ Yes | ✅ Yes | ✅ Yes |

```javascript
// localStorage
localStorage.setItem('theme', 'dark');
localStorage.getItem('theme'); // 'dark'

// sessionStorage
sessionStorage.setItem('step', '2');

// cookies
document.cookie = "user=John; expires=Fri, 31 Dec 2025 23:59:59 GMT";
```

---

## Quick Fire Questions

**Q: What does `<!DOCTYPE html>` do?**
A: Tells the browser to use **HTML5 standards mode**, not quirks mode.

**Q: Can you have multiple `<h1>` tags?**
A: Technically yes, but SEO best practice is **one `<h1>` per page**.

**Q: What's the difference between `<b>` and `<strong>`?**
A: `<b>` is just bold styling. `<strong>` means the text is **important** (semantic).

**Q: What's the difference between `<i>` and `<em>`?**
A: `<i>` is just italic. `<em>` means **emphasis** (semantic, read differently by screen readers).

**Q: What is a void element?**
A: An element that **can't have children** — `<img>`, `<input>`, `<br>`, `<hr>`, `<meta>`, `<link>`.
