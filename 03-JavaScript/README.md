# ⚡ JavaScript Interview Questions & Answers

---

## 1. var vs let vs const — What's the difference?

```javascript
// var — function scoped, hoisted, can be redeclared
var x = 1;
var x = 2; // ✅ no error
function test() {
  if (true) { var y = 10; }
  console.log(y); // 10 — var leaks out of block!
}

// let — block scoped, hoisted but NOT initialized (Temporal Dead Zone)
let a = 1;
let a = 2; // ❌ SyntaxError
{
  let b = 10;
}
console.log(b); // ❌ ReferenceError

// const — block scoped, can't be reassigned
const PI = 3.14;
PI = 3; // ❌ TypeError
const arr = [1, 2, 3];
arr.push(4); // ✅ allowed — you're mutating, not reassigning
```

**Rule:** Always use `const` by default. Use `let` when you need to reassign. Never use `var`.

---

## 2. What is Hoisting?

**🧠 Simple Explanation:** JavaScript moves declarations to the **top of their scope** before running code.

```javascript
// What you write:
console.log(name); // undefined (not error!)
var name = "Alice";

// What JS actually does:
var name; // declaration hoisted
console.log(name); // undefined
name = "Alice"; // assignment stays

// Functions are fully hoisted
greet(); // ✅ "Hello" — works even before declaration
function greet() { console.log("Hello"); }

// let/const — hoisted but in "Temporal Dead Zone"
console.log(age); // ❌ ReferenceError
let age = 25;
```

---

## 3. Closures — The Most Asked JS Question

**🧠 Simple Explanation:** A function that **remembers** the variables from where it was created, even after the outer function finishes.

```javascript
function makeCounter() {
  let count = 0; // this variable is "closed over"

  return function() {
    count++;
    return count;
  };
}

const counter = makeCounter();
counter(); // 1
counter(); // 2
counter(); // 3 — count is preserved!

// Practical use: Private variables
function createBank(initialBalance) {
  let balance = initialBalance; // private!

  return {
    deposit(amount) { balance += amount; },
    withdraw(amount) { balance -= amount; },
    getBalance() { return balance; }
  };
}

const account = createBank(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.balance); // undefined — private!
```

**Classic closure trap in loops:**
```javascript
// ❌ Bug — all print 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}

// ✅ Fix 1 — use let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000); // 0, 1, 2
}

// ✅ Fix 2 — IIFE
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 1000); // 0, 1, 2
  })(i);
}
```

---

## 4. `this` keyword — The Tricky One

**🧠 Simple Explanation:** `this` refers to **who called the function**.

```javascript
// In global scope
console.log(this); // window (browser) or global (Node)

// In object method — this = the object
const user = {
  name: "Alice",
  greet() {
    console.log(this.name); // "Alice"
  }
};
user.greet();

// In regular function — this = undefined (strict mode) or window
function show() {
  console.log(this); // window or undefined
}

// Arrow function — NO own this, inherits from surrounding scope
const obj = {
  name: "Bob",
  greet: () => {
    console.log(this.name); // undefined! Arrow function doesn't bind `this`
  },
  greetCorrect() {
    const inner = () => console.log(this.name); // ✅ "Bob" — inherits from greetCorrect
    inner();
  }
};

// Explicitly setting `this`
function greet() { console.log(this.name); }
greet.call({ name: "Charlie" });    // call — invoke immediately with args
greet.apply({ name: "Dave" }, []);  // apply — same but args as array
const bound = greet.bind({ name: "Eve" }); // bind — returns new function
bound();
```

---

## 5. Prototypes and Prototype Chain

**🧠 Simple Explanation:** Every object has a secret link to another object (its prototype). If a property isn't found, JS looks up the chain.

```javascript
const animal = {
  breathe() { console.log("breathing"); }
};

const dog = Object.create(animal); // dog's prototype = animal
dog.bark = function() { console.log("woof"); };

dog.bark();    // "woof" — found on dog
dog.breathe(); // "breathing" — found on animal (prototype chain!)
dog.toString(); // "[object Object]" — found on Object.prototype

// Class syntax (ES6 — just sugar over prototypes)
class Animal {
  constructor(name) {
    this.name = name;
  }
  breathe() { return `${this.name} is breathing`; }
}

class Dog extends Animal {
  bark() { return `${this.name} says woof`; }
}

const rex = new Dog("Rex");
rex.bark();    // "Rex says woof"
rex.breathe(); // "Rex is breathing"
rex instanceof Dog;    // true
rex instanceof Animal; // true
```

---

## 6. Promises and Async/Await

**🧠 Simple Explanation:** Promise = "I'll get back to you with data... or an error."

```javascript
// Creating a Promise
const fetchData = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) resolve({ name: "Alice" });
    else reject(new Error("Failed to fetch"));
  }, 1000);
});

// Using Promise
fetchData
  .then(data => console.log(data))  // { name: "Alice" }
  .catch(err => console.error(err));

// async/await — cleaner syntax for promises
async function loadUser() {
  try {
    const response = await fetch('/api/user'); // waits here
    const data = await response.json();        // waits here
    console.log(data);
  } catch (error) {
    console.error("Error:", error);
  }
}

// Running promises in parallel
async function loadAll() {
  // Sequential (slow — waits for each)
  const user = await fetchUser();
  const posts = await fetchPosts();

  // Parallel (fast — runs at same time!)
  const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
}

// Promise.allSettled — doesn't fail if one rejects
const results = await Promise.allSettled([p1, p2, p3]);
// results = [{ status: 'fulfilled', value: ... }, { status: 'rejected', reason: ... }]

// Promise.race — resolves with whichever finishes first
const fastest = await Promise.race([fast(), slow()]);
```

---

## 7. Event Loop — How JS handles async

**🧠 Simple Explanation:** JS is single-threaded. The event loop manages what runs when.

```
Call Stack     →  where code runs (synchronous)
Web APIs       →  handles timers, fetch, DOM events (browser)
Callback Queue →  ready callbacks wait here (macrotasks)
Microtask Queue→  promises go here (runs BEFORE callback queue!)
```

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);  // macrotask

Promise.resolve().then(() => console.log("3")); // microtask

console.log("4");

// Output: 1, 4, 3, 2
// Reason: sync first (1,4) → microtasks (3) → macrotasks (2)
```

---

## 8. Spread, Rest, Destructuring

```javascript
// Spread — expand iterable
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

const obj1 = { a: 1 };
const obj2 = { ...obj1, b: 2 }; // { a: 1, b: 2 }

// Merging + overriding
const defaults = { theme: 'light', lang: 'en' };
const userSettings = { theme: 'dark' };
const settings = { ...defaults, ...userSettings }; // { theme: 'dark', lang: 'en' }

// Rest — collect remaining into array
function sum(...numbers) { // numbers = [1, 2, 3, 4]
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10

// Array Destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

// Object Destructuring
const { name, age, city = "Unknown" } = { name: "Alice", age: 25 };
// city uses default "Unknown"

// Rename while destructuring
const { name: userName } = { name: "Bob" };
// userName = "Bob"

// Nested destructuring
const { address: { street } } = { address: { street: "Main St" } };
```

---

## 9. Array Methods — The Big 6

```javascript
const nums = [1, 2, 3, 4, 5];

// map — transform each item → returns NEW array (same length)
nums.map(n => n * 2); // [2, 4, 6, 8, 10]

// filter — keep items that pass test → returns NEW array
nums.filter(n => n > 2); // [3, 4, 5]

// reduce — boil array down to ONE value
nums.reduce((sum, n) => sum + n, 0); // 15

// find — first item that matches (or undefined)
nums.find(n => n > 3); // 4

// some — true if ANY item matches
nums.some(n => n > 4); // true

// every — true if ALL items match
nums.every(n => n > 0); // true

// Chaining
const result = [1, 2, 3, 4, 5, 6]
  .filter(n => n % 2 === 0)   // [2, 4, 6]
  .map(n => n * n)             // [4, 16, 36]
  .reduce((sum, n) => sum + n, 0); // 56
```

---

## 10. Deep Clone vs Shallow Clone

```javascript
const original = { name: "Alice", address: { city: "NYC" } };

// Shallow clone — nested objects still SHARED
const shallow = { ...original };
shallow.name = "Bob";         // ✅ doesn't affect original
shallow.address.city = "LA";  // ❌ DOES affect original!

// Deep clone options
// Method 1: JSON (simple but loses functions, undefined, Dates)
const deep1 = JSON.parse(JSON.stringify(original));

// Method 2: structuredClone (modern, handles more types)
const deep2 = structuredClone(original);

// Method 3: Lodash _.cloneDeep(original)
```

---

## 11. Debounce vs Throttle

**🧠 Simple Explanation:**
- **Debounce** = Wait until the user STOPS doing something, then fire once
- **Throttle** = Fire at most once every X ms, no matter how many times triggered

```javascript
// Debounce — for search inputs, resize
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce((query) => fetchResults(query), 300);
inputEl.addEventListener('input', e => search(e.target.value));
// Only fires 300ms AFTER user stops typing

// Throttle — for scroll events, button spam
function throttle(fn, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

const handleScroll = throttle(() => updateHeader(), 100);
window.addEventListener('scroll', handleScroll);
// Fires at most once per 100ms
```

---

## 12. `==` vs `===`

```javascript
// == (loose equality) — converts types before comparing
0 == false    // true  (false becomes 0)
"" == false   // true
null == undefined // true
1 == "1"      // true

// === (strict equality) — no type conversion
0 === false   // false
1 === "1"     // false
null === undefined // false

// Always use === unless you specifically need type coercion
```

---

## Quick Fire Questions

**Q: What is IIFE?**
```javascript
(function() {
  // Immediately Invoked Function Expression
  // Creates its own scope — used to avoid polluting global
})();
```

**Q: What is the difference between `null` and `undefined`?**
- `undefined` = variable declared but not assigned
- `null` = intentionally set to "no value"

**Q: What is `typeof null`?**
A: `"object"` — this is a famous JS bug that can't be fixed for backward compatibility.

**Q: What is optional chaining (`?.`)?**
```javascript
const street = user?.address?.street; // undefined instead of throwing error
const fn = obj?.method?.(); // safely call if method exists
```

**Q: What is nullish coalescing (`??`)?**
```javascript
const name = user.name ?? "Guest"; // uses "Guest" only if name is null/undefined
// Different from || which also triggers on 0, "", false
const count = 0 ?? 10; // 0 (correct)
const count2 = 0 || 10; // 10 (wrong for count!)
```
