# ⚛️ React Interview Questions & Answers

---

## 1. What is the Virtual DOM and how does React use it?

**🧠 Simple Explanation:**
The real DOM is slow to update. React creates a **lightweight copy** of the DOM in memory (Virtual DOM), compares changes, and only updates what actually changed.

```
Step 1: React builds Virtual DOM (JS object tree)
Step 2: State changes → new Virtual DOM created
Step 3: React DIFFS old vs new Virtual DOM (reconciliation)
Step 4: Only changed parts are updated in the REAL DOM
```

This process is called **reconciliation**. The comparison algorithm is called the **diffing algorithm**.

---

## 2. useState — Complete Guide

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // initial value = 0

  // ✅ Correct — using updater function (avoids stale state)
  const increment = () => setCount(prev => prev + 1);

  // ❌ Wrong for batched updates
  const doubleIncrement = () => {
    setCount(count + 1); // both use old count!
    setCount(count + 1); // result: count + 1, not count + 2
  };

  // ✅ Correct — updater function queues correctly
  const correctDouble = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1); // result: count + 2
  };

  return <button onClick={increment}>{count}</button>;
}

// Object state
const [user, setUser] = useState({ name: "Alice", age: 25 });

// ✅ Spread to merge (setState doesn't auto-merge for hooks!)
setUser(prev => ({ ...prev, age: 26 }));
// ❌ This REPLACES the whole object
setUser({ age: 26 }); // name is gone!
```

---

## 3. useEffect — Complete Guide

**🧠 Think of it as:** "Do this side effect when something changes."

```jsx
import { useEffect } from 'react';

// Run after EVERY render
useEffect(() => {
  console.log("I run after every render");
});

// Run only ONCE (on mount)
useEffect(() => {
  console.log("I run once when component mounts");
}, []); // empty dependency array

// Run when specific values change
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // runs when `count` changes

// Cleanup function — runs before next effect OR on unmount
useEffect(() => {
  const timer = setInterval(() => tick(), 1000);

  return () => {
    clearInterval(timer); // cleanup! prevents memory leaks
  };
}, []);

// Fetching data
useEffect(() => {
  let cancelled = false;

  async function fetchUser() {
    const data = await fetch('/api/user').then(r => r.json());
    if (!cancelled) setUser(data); // avoid state update after unmount
  }

  fetchUser();
  return () => { cancelled = true; };
}, [userId]);
```

---

## 4. useMemo and useCallback — Performance Optimization

```jsx
// useMemo — memoize EXPENSIVE CALCULATIONS
const expensiveResult = useMemo(() => {
  return items.filter(i => i.active).sort().slice(0, 100);
}, [items]); // only recalculates when `items` changes

// useCallback — memoize FUNCTIONS (prevents child re-renders)
const handleSubmit = useCallback((data) => {
  api.post('/submit', data);
}, [api]); // same function reference unless `api` changes

// When to use:
// useMemo — filtering/sorting large lists, complex math
// useCallback — passing callbacks to optimized child components (with React.memo)

// React.memo — prevents re-render if props didn't change
const Button = React.memo(({ onClick, label }) => {
  console.log("Button rendered");
  return <button onClick={onClick}>{label}</button>;
});

// Without useCallback, Button re-renders every parent render
// With useCallback, Button only re-renders when dependencies change
```

---

## 5. useRef — Beyond DOM Access

```jsx
// Use 1: Accessing DOM elements
const inputRef = useRef(null);
<input ref={inputRef} />
inputRef.current.focus(); // programmatically focus

// Use 2: Storing mutable values WITHOUT triggering re-render
const renderCount = useRef(0);
useEffect(() => {
  renderCount.current++; // doesn't cause re-render!
  console.log(`Rendered ${renderCount.current} times`);
});

// Use 3: Previous value pattern
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current; // previous value
}

// Use 4: Avoiding stale closures
const latestCallback = useRef(callback);
useEffect(() => {
  latestCallback.current = callback;
}, [callback]);
```

---

## 6. Custom Hooks

**🧠 Simple Explanation:** Extract reusable stateful logic into a function starting with `use`.

```jsx
// useFetch — reusable data fetching
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(r => r.json())
      .then(data => { setData(data); setLoading(false); })
      .catch(err => { setError(err); setLoading(false); });
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserProfile({ id }) {
  const { data: user, loading, error } = useFetch(`/api/users/${id}`);

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  return <div>{user.name}</div>;
}

// useLocalStorage hook
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  const setAndStore = (newValue) => {
    setValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  };

  return [value, setAndStore];
}
```

---

## 7. Context API — Avoiding Prop Drilling

```jsx
// Create context
const ThemeContext = createContext('light');
const UserContext = createContext(null);

// Provider — wrap your app (or part of it)
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={currentUser}>
        <Router />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// Consumer — use in any nested component without prop drilling
function Navbar() {
  const { theme, setTheme } = useContext(ThemeContext);
  const user = useContext(UserContext);

  return (
    <nav className={theme}>
      <span>{user.name}</span>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle
      </button>
    </nav>
  );
}

// ⚠️ Context re-renders ALL consumers when value changes
// Split contexts to minimize re-renders
// Use useMemo to stabilize value object
const value = useMemo(() => ({ theme, setTheme }), [theme]);
```

---

## 8. useReducer — Complex State Logic

```jsx
// Great for: multiple related state values, complex update logic
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step };
    case 'DECREMENT':
      return { ...state, count: state.count - state.step };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    case 'RESET':
      return initialState;
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <input
        onChange={e => dispatch({ type: 'SET_STEP', payload: +e.target.value })}
      />
    </div>
  );
}
```

---

## 9. React Component Lifecycle

```
MOUNT:
  constructor / useState init
  render
  DOM update
  useEffect(() => {...}, [])  ← "componentDidMount"

UPDATE (props/state change):
  render
  DOM update
  useEffect cleanup (previous)
  useEffect(() => {...}, [dep])  ← "componentDidUpdate"

UNMOUNT:
  useEffect cleanup function  ← "componentWillUnmount"
```

```jsx
useEffect(() => {
  console.log("MOUNT or UPDATE");
  return () => {
    console.log("CLEANUP (before next effect or unmount)");
  };
}, [dependency]);
```

---

## 10. React Performance Optimization

```jsx
// 1. React.memo — skip re-render if props same
const HeavyComponent = React.memo(({ data }) => {
  return <ExpensiveRender data={data} />;
});

// 2. useMemo — cache expensive computation
const sortedList = useMemo(() => 
  [...items].sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// 3. useCallback — stable function reference
const handleClick = useCallback(() => doSomething(id), [id]);

// 4. Code splitting — lazy load components
const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Dashboard />
    </Suspense>
  );
}

// 5. List keys — help React track items
// ❌ Bad — using index as key (causes issues with reordering)
items.map((item, index) => <Item key={index} />)

// ✅ Good — use stable unique ID
items.map(item => <Item key={item.id} />)

// 6. Avoid inline object/function creation in JSX
// ❌ Creates new object every render
<Component style={{ color: 'red' }} />

// ✅ Define outside component or useMemo
const style = { color: 'red' };
<Component style={style} />
```

---

## 11. Common React Interview Patterns

### Error Boundary
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logError(error, info);
  }

  render() {
    if (this.state.hasError) return <h1>Something went wrong</h1>;
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

### Controlled vs Uncontrolled Components
```jsx
// Controlled — React controls the value
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />

// Uncontrolled — DOM controls the value
const ref = useRef();
<input ref={ref} defaultValue="hello" />
// Get value: ref.current.value
```

---

## Quick Fire Questions

**Q: What is reconciliation?**
A: The process React uses to compare old and new Virtual DOM trees and determine the minimal DOM updates needed.

**Q: What is the key prop for?**
A: Helps React identify which items changed in a list. Must be unique and stable (not index).

**Q: Can you call hooks conditionally?**
A: ❌ No. Hooks must be called in the same order every render (no inside if/loop/nested functions).

**Q: What's the difference between state and props?**
- Props → passed from parent, read-only in child
- State → owned by component, can be changed, triggers re-render

**Q: What is lifting state up?**
A: Moving state to the closest common ancestor so multiple children can share it.
