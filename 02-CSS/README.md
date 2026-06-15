# 🎨 CSS Interview Questions & Answers

---

## 1. What is the CSS Box Model?

**🧠 Simple Explanation:**
Every HTML element is a box. The box has 4 layers:

```
┌─────────────────────────────┐
│         MARGIN              │  ← Space OUTSIDE the border
│  ┌───────────────────────┐  │
│  │       BORDER          │  │  ← The border line
│  │  ┌─────────────────┐  │  │
│  │  │    PADDING      │  │  │  ← Space INSIDE the border
│  │  │  ┌───────────┐  │  │  │
│  │  │  │  CONTENT  │  │  │  │  ← Your actual text/image
│  │  │  └───────────┘  │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

```css
.box {
  width: 200px;       /* content width */
  padding: 20px;      /* space inside */
  border: 2px solid black;
  margin: 10px;       /* space outside */
}
```

**Total actual width = width + padding×2 + border×2**
= 200 + 40 + 4 = **244px**

### `box-sizing: border-box` — The Fix

```css
* {
  box-sizing: border-box; /* padding + border are INCLUDED in width */
}
/* Now width: 200px means the TOTAL is 200px — much easier! */
```

---

## 2. CSS Specificity — Which style wins?

When two CSS rules target the same element, the **more specific one wins**.

**Specificity score (a, b, c, d):**

| Selector | Score |
|----------|-------|
| `!important` | Always wins (avoid!) |
| Inline style (`style=""`) | (1,0,0,0) |
| ID (`#id`) | (0,1,0,0) |
| Class (`.class`), attribute, pseudo-class | (0,0,1,0) |
| Element (`div`, `p`), pseudo-element | (0,0,0,1) |

```css
p { color: black; }           /* 0,0,0,1 */
.text { color: blue; }        /* 0,0,1,0 — wins over above */
#main { color: green; }       /* 0,1,0,0 — wins over above */
p { color: red !important; }  /* wins over everything (avoid!) */
```

**Trick question:** What color is `<p class="text" id="main">`?
Answer: **green** (ID wins)

---

## 3. Flexbox — Complete Guide

**🧠 Think of it as:** A row (or column) where items share space intelligently.

```css
.container {
  display: flex;
  flex-direction: row;       /* row | column | row-reverse | column-reverse */
  justify-content: center;   /* main axis: start|center|end|space-between|space-around */
  align-items: center;       /* cross axis: stretch|center|start|end */
  flex-wrap: wrap;           /* wrap to next line if items overflow */
  gap: 16px;                 /* space between items */
}

.item {
  flex: 1;          /* grow and shrink equally */
  flex: 0 0 200px;  /* don't grow, don't shrink, stay 200px */
  align-self: flex-end; /* override parent's align-items for this item */
  order: 2;         /* change visual order */
}
```

**Classic centering with flex:**
```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

---

## 4. CSS Grid — Complete Guide

**🧠 Think of it as:** A table — rows AND columns.

```css
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 1fr;  /* 3 columns */
  grid-template-rows: auto;
  gap: 16px;
}

/* Responsive grid without media queries */
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
}

/* Placing items */
.item {
  grid-column: 1 / 3;    /* span from column 1 to 3 */
  grid-row: 1 / 2;
}
```

### Grid vs Flexbox — When to use which?

| Use Case | Use |
|----------|-----|
| 1D layout (row OR column) | Flexbox |
| 2D layout (rows AND columns) | Grid |
| Navigation bar | Flexbox |
| Page layout | Grid |
| Card row | Either |

---

## 5. Position property explained

```css
/* static (default) — normal document flow, top/left/etc. have NO effect */
.a { position: static; }

/* relative — moves RELATIVE to its normal position, space is still reserved */
.b { position: relative; top: 10px; left: 20px; }

/* absolute — removed from flow, positions relative to nearest non-static ancestor */
.c { position: absolute; top: 0; right: 0; }

/* fixed — removed from flow, always relative to VIEWPORT (stays during scroll) */
.d { position: fixed; bottom: 20px; right: 20px; }

/* sticky — acts like relative UNTIL you scroll past it, then acts like fixed */
.e { position: sticky; top: 0; } /* great for sticky headers */
```

---

## 6. What is a CSS pseudo-class vs pseudo-element?

**Pseudo-class** → targets element in a specific **state**
```css
a:hover { color: red; }       /* when mouse is over it */
input:focus { outline: blue; } /* when focused */
li:first-child { }             /* first child */
li:nth-child(2n) { }          /* every even child */
input:checked { }             /* checked checkbox */
```

**Pseudo-element** → targets a **part of** an element (uses `::` double colon)
```css
p::first-line { font-weight: bold; }
p::first-letter { font-size: 2em; }

/* Add content before/after without HTML */
.required::after {
  content: ' *';
  color: red;
}
```

---

## 7. CSS Variables (Custom Properties)

```css
/* Define on :root to make global */
:root {
  --primary: #3b82f6;
  --radius: 8px;
  --font-size-lg: 1.25rem;
}

.button {
  background: var(--primary);
  border-radius: var(--radius);
  font-size: var(--font-size-lg);
}

/* Override in a scope */
.dark-theme {
  --primary: #60a5fa;
}

/* With fallback */
color: var(--text-color, black);
```

---

## 8. Media Queries — Responsive Design

```css
/* Mobile first approach (recommended) */
.card {
  width: 100%;  /* mobile: full width */
}

@media (min-width: 768px) {
  .card { width: 50%; }   /* tablet */
}

@media (min-width: 1024px) {
  .card { width: 33.33%; } /* desktop */
}

/* Common breakpoints */
/* 480px  — small phones */
/* 768px  — tablets */
/* 1024px — laptops */
/* 1280px — desktops */

/* Other media features */
@media (prefers-color-scheme: dark) { }
@media (prefers-reduced-motion: reduce) { }
@media print { }
```

---

## 9. CSS Animations vs Transitions

**Transitions** — smooth change between two states
```css
.button {
  background: blue;
  transition: background 0.3s ease, transform 0.2s ease;
}
.button:hover {
  background: darkblue;
  transform: scale(1.05);
}
```

**Animations** — complex, multi-step, can auto-play
```css
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.modal {
  animation: fadeIn 0.4s ease forwards;
}
```

### `transform` properties (GPU-accelerated, fast!)
```css
transform: translate(50px, 100px);  /* move */
transform: scale(1.5);              /* resize */
transform: rotate(45deg);           /* rotate */
transform: skew(10deg);             /* skew */
/* Combine: */
transform: translate(-50%, -50%) rotate(45deg) scale(0.8);
```

---

## 10. Z-index and Stacking Context

`z-index` controls which element appears **on top**.

```css
.modal    { z-index: 1000; }
.dropdown { z-index: 500; }
.header   { z-index: 100; }
.content  { z-index: 1; }
```

**⚠️ Gotcha:** z-index only works on elements with `position` set (not `static`).

**Stacking context** — created by:
- `position: relative/absolute/fixed` + `z-index` not auto
- `opacity` less than 1
- `transform`, `filter`, `will-change`

---

## Quick Fire Questions

**Q: How do you center a div horizontally and vertically?**
```css
/* Method 1: Flexbox (recommended) */
.parent { display: flex; justify-content: center; align-items: center; }

/* Method 2: Grid */
.parent { display: grid; place-items: center; }

/* Method 3: Absolute + transform */
.child { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
```

**Q: What is `em` vs `rem`?**
- `em` → relative to the **parent** element's font-size
- `rem` → relative to the **root** (`<html>`) font-size (more predictable, preferred)

**Q: What is `display: none` vs `visibility: hidden` vs `opacity: 0`?**
| | Visible | Takes space | Events |
|--|---------|-------------|--------|
| `display: none` | ❌ | ❌ | ❌ |
| `visibility: hidden` | ❌ | ✅ | ❌ |
| `opacity: 0` | ❌ | ✅ | ✅ |

**Q: What is BEM naming?**
```css
/* Block__Element--Modifier */
.card { }
.card__title { }
.card__button { }
.card__button--disabled { }
```
