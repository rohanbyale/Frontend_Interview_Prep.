# ⚡ Frontend Performance Interview Questions & Answers

---

## 1. Core Web Vitals — What interviewers love asking about

Google's 3 key metrics for user experience:

| Metric | What it measures | Good | Poor |
|--------|-----------------|------|------|
| **LCP** (Largest Contentful Paint) | Loading speed | < 2.5s | > 4s |
| **FID** (First Input Delay) / **INP** | Interactivity | < 100ms | > 300ms |
| **CLS** (Cumulative Layout Shift) | Visual stability | < 0.1 | > 0.25 |

```
LCP — How fast is the MAIN content visible?
     → Optimize: images, fonts, server speed, render-blocking resources

FID/INP — How fast does the page respond to clicks/typing?
     → Optimize: reduce JS execution time, break up long tasks

CLS — Does content jump around while loading?
     → Optimize: size images/videos, don't insert content above existing
```

---

## 2. How to optimize images (huge impact!)

```html
<!-- 1. Always specify width and height (prevents CLS) -->
<img src="photo.jpg" width="800" height="600" alt="...">

<!-- 2. Lazy load images below the fold -->
<img src="photo.jpg" loading="lazy" alt="...">

<!-- 3. Use modern formats -->
<picture>
  <source srcset="photo.avif" type="image/avif">
  <source srcset="photo.webp" type="image/webp">
  <img src="photo.jpg" alt="...">
</picture>

<!-- 4. Responsive images — load right size for screen -->
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 900px) 800px, 1200px"
  alt="..."
>
```

**Format file sizes (for same quality):**
- JPEG: 100KB baseline
- WebP: ~60KB (40% smaller)
- AVIF: ~40KB (60% smaller)

---

## 3. Critical Rendering Path

```
1. HTML → DOM tree
2. CSS → CSSOM tree
3. DOM + CSSOM → Render Tree
4. Layout (calculate positions)
5. Paint (draw pixels)
6. Composite (layers to screen)
```

**Optimization tips:**
```html
<!-- 1. Put CSS in <head> (don't block rendering) -->
<link rel="stylesheet" href="style.css">

<!-- 2. Defer non-critical JS -->
<script defer src="app.js"></script>
<script async src="analytics.js"></script>

<!-- 3. Inline critical CSS -->
<style>
  /* Only the CSS needed for above-the-fold content */
  body { margin: 0; }
  .header { background: #333; }
</style>
<link rel="stylesheet" href="rest.css" media="print" onload="this.media='all'">
```

---

## 4. Code Splitting and Lazy Loading

```javascript
// Without code splitting — ONE huge JS bundle (bad!)
import Dashboard from './Dashboard';
import Analytics from './Analytics';
import Settings from './Settings';

// With code splitting — loads only what's needed (good!)
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));
const Analytics = lazy(() => import('./Analytics'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Route path="/dashboard" component={Dashboard} />
      <Route path="/analytics" component={Analytics} />
    </Suspense>
  );
}

// Dynamic imports in vanilla JS
button.addEventListener('click', async () => {
  const { openModal } = await import('./modal.js'); // loads only on click!
  openModal();
});
```

---

## 5. Caching Strategies

```
Browser Cache Headers:
├── Cache-Control: max-age=31536000 (1 year) → for versioned files (bundle.abc123.js)
├── Cache-Control: no-cache → always revalidate (HTML pages)
├── Cache-Control: no-store → never cache (sensitive data)
└── ETag: "abc123" → version tag for revalidation
```

```javascript
// Service Worker caching strategies

// Cache First — serve from cache, update in background (for static assets)
async function cacheFirst(request) {
  const cached = await caches.match(request);
  return cached ?? fetch(request);
}

// Network First — try network, fallback to cache (for API calls)
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    cache.put(request, response.clone());
    return response;
  } catch {
    return caches.match(request);
  }
}

// Stale While Revalidate — serve cached immediately, update in background
async function staleWhileRevalidate(request) {
  const cached = await caches.match(request);
  const networkPromise = fetch(request).then(r => {
    cache.put(request, r.clone());
    return r;
  });
  return cached ?? networkPromise;
}
```

---

## 6. JavaScript Performance

```javascript
// 1. Avoid layout thrashing (reading then writing DOM alternately)
// ❌ Bad — browser reflows multiple times
for (let i = 0; i < items.length; i++) {
  const height = items[i].offsetHeight; // read (forces layout)
  items[i].style.height = height + 10 + 'px'; // write
}

// ✅ Good — batch reads, then writes
const heights = items.map(el => el.offsetHeight); // all reads
items.forEach((el, i) => el.style.height = heights[i] + 10 + 'px'); // all writes

// 2. Use requestAnimationFrame for visual updates
function animate() {
  // Update DOM
  el.style.transform = `translateX(${x}px)`;
  requestAnimationFrame(animate); // synced with screen refresh
}

// 3. Use Web Workers for heavy computation
const worker = new Worker('./heavy-calc.js');
worker.postMessage({ data: bigArray });
worker.onmessage = (e) => setResult(e.data); // doesn't block UI!

// 4. Virtualize long lists (only render visible items)
// Libraries: react-window, react-virtual
// Instead of rendering 10,000 items, render only the ~20 visible
```

---

## 7. Network Performance

```html
<!-- Preconnect to external origins -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://api.example.com">

<!-- Prefetch — load resource for next page navigation -->
<link rel="prefetch" href="/next-page.js">

<!-- Preload — load resource needed for current page ASAP -->
<link rel="preload" href="hero-font.woff2" as="font" crossorigin>
<link rel="preload" href="hero.jpg" as="image">
```

```javascript
// Compression — always enable gzip/brotli on server
// Brotli is ~20% better than gzip

// HTTP/2 — multiplexing (send multiple requests at once)
// HTTP/3 (QUIC) — even better for mobile/lossy networks

// CDN — serve assets from servers close to users
// Static assets: images, CSS, JS → always use CDN
```

---

## 8. React-Specific Performance

```jsx
// 1. Avoid unnecessary re-renders
const Component = React.memo(({ data }) => <ExpensiveRender data={data} />);

// 2. Virtualize long lists
import { FixedSizeList } from 'react-window';
<FixedSizeList height={600} itemCount={10000} itemSize={50}>
  {({ index, style }) => <div style={style}>Item {index}</div>}
</FixedSizeList>

// 3. useDeferredValue — defer low-priority updates
const deferredQuery = useDeferredValue(query); // search results can lag
const results = useMemo(() => filterItems(deferredQuery), [deferredQuery]);

// 4. useTransition — mark updates as non-urgent
const [isPending, startTransition] = useTransition();
startTransition(() => {
  setFilteredList(heavyFilter(data)); // doesn't block typing
});

// 5. Profiler
<Profiler id="MyComponent" onRender={(id, phase, duration) => {
  console.log(`${id} took ${duration}ms to ${phase}`);
}}>
  <MyComponent />
</Profiler>
```

---

## Quick Fire Questions

**Q: What is TTFB?**
A: Time To First Byte — how long until the browser receives the first byte from the server. Optimized by fast servers, CDN, and caching.

**Q: What is tree shaking?**
A: Removing unused code from your bundle. Bundlers like Webpack/Rollup analyze imports and remove code that's never used.

**Q: What's the difference between `paint` and `composite`?**
- **Paint** → drawing pixels for each layer (expensive)
- **Composite** → layering those painted layers together (cheap, GPU)
- Use `transform` and `opacity` for animations — they skip paint, go straight to composite!

**Q: What is prefetching vs preloading?**
- **Preload** → I'll need this resource VERY SOON on this page
- **Prefetch** → I might need this for the NEXT navigation
