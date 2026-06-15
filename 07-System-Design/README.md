# 🏛️ Frontend System Design Interview Questions

---

## 1. Design a Search Autocomplete Component

**What the interviewer wants:** Component architecture, performance, accessibility, edge cases.

```
Requirements:
- Show suggestions as user types
- Debounce API calls
- Keyboard navigation (↑↓ Enter Escape)
- Loading/error states
- Accessible (screen readers)
- Cache results
```

```jsx
function SearchAutocomplete({ onSelect }) {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [loading, setLoading] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const cache = useRef({});

  // Debounced search
  const debouncedSearch = useCallback(
    debounce(async (q) => {
      if (!q.trim()) { setSuggestions([]); return; }

      // Cache check
      if (cache.current[q]) {
        setSuggestions(cache.current[q]);
        return;
      }

      setLoading(true);
      try {
        const data = await searchAPI(q);
        cache.current[q] = data;
        setSuggestions(data);
      } catch (e) {
        setSuggestions([]);
      } finally {
        setLoading(false);
      }
    }, 300),
    []
  );

  // Keyboard navigation
  const handleKeyDown = (e) => {
    if (e.key === 'ArrowDown') setActiveIndex(i => Math.min(i + 1, suggestions.length - 1));
    if (e.key === 'ArrowUp') setActiveIndex(i => Math.max(i - 1, -1));
    if (e.key === 'Enter' && activeIndex >= 0) onSelect(suggestions[activeIndex]);
    if (e.key === 'Escape') setSuggestions([]);
  };

  return (
    <div role="combobox" aria-expanded={suggestions.length > 0}>
      <input
        aria-autocomplete="list"
        aria-controls="suggestions-list"
        value={query}
        onChange={e => { setQuery(e.target.value); debouncedSearch(e.target.value); }}
        onKeyDown={handleKeyDown}
      />
      {loading && <Spinner aria-label="Loading suggestions" />}
      <ul id="suggestions-list" role="listbox">
        {suggestions.map((item, i) => (
          <li
            key={item.id}
            role="option"
            aria-selected={i === activeIndex}
            onClick={() => onSelect(item)}
          >
            {item.label}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 2. Design an Infinite Scroll List

```jsx
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);
  const loaderRef = useRef(null);

  // Intersection Observer — fires when loader enters viewport
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasMore && !loading) {
          loadMore();
        }
      },
      { threshold: 0.1 }
    );

    if (loaderRef.current) observer.observe(loaderRef.current);
    return () => observer.disconnect();
  }, [hasMore, loading]);

  const loadMore = async () => {
    setLoading(true);
    const data = await fetchItems(page);

    if (data.length === 0) {
      setHasMore(false);
    } else {
      setItems(prev => [...prev, ...data]);
      setPage(p => p + 1);
    }
    setLoading(false);
  };

  return (
    <ul>
      {items.map(item => <ListItem key={item.id} item={item} />)}
      {hasMore && <li ref={loaderRef}>{loading ? <Spinner /> : null}</li>}
      {!hasMore && <li>No more items</li>}
    </ul>
  );
}
```

---

## 3. Design a Toast Notification System

```jsx
// Context-based — accessible from anywhere in the app
const ToastContext = createContext(null);

function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);

  const addToast = useCallback(({ message, type = 'info', duration = 3000 }) => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);
    setTimeout(() => removeToast(id), duration);
  }, []);

  const removeToast = useCallback((id) => {
    setToasts(prev => prev.filter(t => t.id !== id));
  }, []);

  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      <div aria-live="polite" aria-label="Notifications">
        {toasts.map(toast => (
          <Toast key={toast.id} {...toast} onClose={() => removeToast(toast.id)} />
        ))}
      </div>
    </ToastContext.Provider>
  );
}

// Usage anywhere in the app
function SomeComponent() {
  const { addToast } = useContext(ToastContext);
  return (
    <button onClick={() => addToast({ message: "Saved!", type: "success" })}>
      Save
    </button>
  );
}
```

---

## 4. State Management — When to use what?

```
useState         → simple local state (toggle, input value)
useReducer       → complex local state (form with many fields)
Context API      → global but rarely changing (theme, user, language)
Zustand/Redux    → global frequently changing (cart, notifications)
React Query/SWR  → server state (cached API data, loading/error states)
URL state        → shareable state (filters, pagination, selected tab)
```

```javascript
// URL state — users can share/bookmark filtered views
const [searchParams, setSearchParams] = useSearchParams();
const filter = searchParams.get('filter') || 'all';
// URL: /products?filter=active&sort=name
```

---

## 5. Design a Multi-Step Form

```jsx
// State machine approach — most robust
const STEPS = ['personal', 'address', 'payment', 'review'];

function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState({});
  const [errors, setErrors] = useState({});

  const updateData = (stepData) => {
    setFormData(prev => ({ ...prev, ...stepData }));
  };

  const validateStep = (step, data) => {
    // step-specific validation
    const stepErrors = validators[step](data);
    setErrors(stepErrors);
    return Object.keys(stepErrors).length === 0;
  };

  const next = (stepData) => {
    updateData(stepData);
    if (validateStep(STEPS[currentStep], stepData)) {
      setCurrentStep(s => Math.min(s + 1, STEPS.length - 1));
    }
  };

  const back = () => setCurrentStep(s => Math.max(s - 1, 0));

  const submit = async () => {
    await api.submitForm(formData);
  };

  const StepComponent = stepComponents[STEPS[currentStep]];

  return (
    <div>
      {/* Progress indicator */}
      <ProgressBar steps={STEPS} current={currentStep} />

      {/* Current step */}
      <StepComponent
        data={formData}
        errors={errors}
        onNext={next}
        onBack={currentStep > 0 ? back : null}
        onSubmit={currentStep === STEPS.length - 1 ? submit : null}
      />
    </div>
  );
}
```

---

## 6. Micro-Frontend Architecture

**🧠 Simple Explanation:** Instead of ONE giant frontend, you split into multiple independent mini-apps that compose together.

```
Traditional Monolith:              Micro-Frontend:
┌─────────────────┐               ┌──────┐ ┌──────┐ ┌──────┐
│                 │               │ Team │ │ Team │ │ Team │
│  ONE big app    │    vs         │  A   │ │  B   │ │  C   │
│  (one team)     │               │(nav) │ │(cart)│ │(pdp) │
│                 │               └──────┘ └──────┘ └──────┘
└─────────────────┘                    ↓ Compose in Shell App
```

**Implementation options:**
- **iframes** — isolation but poor UX
- **Module Federation** (Webpack 5) — share code between apps at runtime
- **Web Components** — framework-agnostic custom elements

```javascript
// Webpack Module Federation — share components across apps
// Host app webpack config:
new ModuleFederationPlugin({
  name: 'shell',
  remotes: {
    nav: 'nav@https://nav.example.com/remoteEntry.js',
    cart: 'cart@https://cart.example.com/remoteEntry.js',
  }
});

// Use in host:
const NavBar = lazy(() => import('nav/NavBar'));
const CartWidget = lazy(() => import('cart/CartWidget'));
```

---

## 7. Accessibility (a11y) Design

```jsx
// Checklist for every component:

// 1. Semantic HTML first
<button onClick={toggle}>Menu</button>  // ✅
<div onClick={toggle}>Menu</div>        // ❌ not keyboard accessible

// 2. ARIA for custom components
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-desc"
>
  <h2 id="dialog-title">Confirm Delete</h2>
  <p id="dialog-desc">This action cannot be undone.</p>
</div>

// 3. Focus management — trap focus in modals
function Modal({ onClose }) {
  const firstFocusRef = useRef(null);

  useEffect(() => {
    firstFocusRef.current?.focus(); // focus first element on open
    return () => previousFocusEl.focus(); // restore focus on close
  }, []);
}

// 4. Keyboard navigation
onKeyDown={(e) => {
  if (e.key === 'Enter' || e.key === ' ') handleClick();
  if (e.key === 'Escape') handleClose();
}}

// 5. Skip links (for keyboard users)
<a href="#main-content" className="skip-link">Skip to main content</a>

// 6. Color contrast — 4.5:1 ratio for normal text, 3:1 for large text
```
