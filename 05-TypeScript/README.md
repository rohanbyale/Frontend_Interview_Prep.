# 🔷 TypeScript Interview Questions & Answers

---

## 1. Type vs Interface — What's the difference?

```typescript
// Interface — for objects/classes, can be extended and merged
interface User {
  id: number;
  name: string;
}
interface User {
  email: string; // declaration merging — adds to existing interface!
}
interface Admin extends User {
  role: string;
}

// Type — more flexible, can represent anything
type ID = number | string;           // unions
type Point = [number, number];        // tuples
type Callback = () => void;           // functions
type User = { id: number; name: string }; // objects too

// Extending types (intersection)
type Admin = User & { role: string };

// 🧠 Rule of thumb:
// Use interface for object shapes (especially public APIs)
// Use type for unions, intersections, and primitives
```

---

## 2. Union and Intersection Types

```typescript
// Union ( | ) — can be ONE of these types
type ID = number | string;
type Status = "pending" | "active" | "inactive"; // string literal union

let status: Status = "active"; // ✅
let status2: Status = "deleted"; // ❌ Error

function printId(id: number | string) {
  // Narrowing — TypeScript knows which type based on condition
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // TypeScript knows it's string here
  } else {
    console.log(id.toFixed(2)); // TypeScript knows it's number here
  }
}

// Intersection ( & ) — must have ALL of these types
type Employee = Person & { company: string; salary: number };
// Must have all Person fields AND company AND salary
```

---

## 3. Generics — The Superpower of TypeScript

**🧠 Simple Explanation:** A placeholder for a type that you fill in when you use it.

```typescript
// Without generics — not reusable
function getFirst(arr: number[]): number { return arr[0]; }

// With generics — works for any type!
function getFirst<T>(arr: T[]): T { return arr[0]; }

getFirst<number>([1, 2, 3]);   // returns number
getFirst<string>(["a", "b"]);  // returns string
getFirst([true, false]);        // TypeScript infers boolean

// Generic with constraint
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}
getLength("hello");    // ✅ strings have length
getLength([1, 2, 3]); // ✅ arrays have length
getLength(42);         // ❌ numbers don't have length

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: "Alice" },
  status: 200,
  message: "OK"
};

// Generic class
class Stack<T> {
  private items: T[] = [];
  push(item: T) { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
  peek(): T | undefined { return this.items[this.items.length - 1]; }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push("hello"); // ❌ Error
```

---

## 4. Utility Types — Built-in TypeScript Magic

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // optional
}

// Partial — all properties become optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number }
// Use case: update functions where you only send changed fields

// Required — all properties become required
type RequiredUser = Required<User>;
// { id: number; name: string; email: string; age: number }

// Readonly — can't be mutated
type ReadonlyUser = Readonly<User>;
const user: ReadonlyUser = { id: 1, name: "Alice", email: "a@a.com" };
user.name = "Bob"; // ❌ Error

// Pick — select specific properties
type UserSummary = Pick<User, 'id' | 'name'>;
// { id: number; name: string }

// Omit — exclude specific properties
type PublicUser = Omit<User, 'email' | 'age'>;
// { id: number; name: string }

// Record — create object type with specific keys/values
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;
const roles: UserRoles = { alice: 'admin', bob: 'user' };

// ReturnType — extract return type of a function
function createUser() { return { id: 1, name: "Alice" }; }
type CreatedUser = ReturnType<typeof createUser>;
// { id: number; name: string }

// Parameters — extract parameter types
function login(username: string, password: string) {}
type LoginParams = Parameters<typeof login>;
// [string, string]
```

---

## 5. Type Narrowing

```typescript
// TypeScript narrows the type based on your checks

function process(value: string | number | null) {
  // typeof narrowing
  if (typeof value === "string") {
    value.toUpperCase(); // ✅ TypeScript knows it's string
  }

  // null check narrowing
  if (value !== null) {
    value; // string | number here
  }

  // instanceof narrowing
  if (value instanceof Date) {
    value.toISOString();
  }
}

// Discriminated unions — most common pattern
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // TS knows radius exists
    case "square":
      return shape.size ** 2;             // TS knows size exists
    case "rectangle":
      return shape.width * shape.height;  // TS knows width & height
  }
}
```

---

## 6. Enums

```typescript
// Numeric enum (default)
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}
Direction.Up; // 0

// String enum (preferred — more readable in logs/debugging)
enum Status {
  Pending = "PENDING",
  Active = "ACTIVE",
  Inactive = "INACTIVE"
}
Status.Active; // "ACTIVE"

// Alternative: const object + typeof (avoids enum quirks)
const STATUS = {
  PENDING: "pending",
  ACTIVE: "active",
  INACTIVE: "inactive"
} as const;

type Status = typeof STATUS[keyof typeof STATUS]; // "pending" | "active" | "inactive"
```

---

## 7. Type Assertions and Guards

```typescript
// Type assertion — "I know better than you, TypeScript"
const input = document.getElementById("name") as HTMLInputElement;
input.value; // ✅ TypeScript knows it's an input

// Non-null assertion — "trust me, it's not null"
const el = document.getElementById("app")!; // ! = definitely not null

// Type guard — custom function to narrow types
interface Cat { meow(): void; }
interface Dog { bark(): void; }

function isCat(animal: Cat | Dog): animal is Cat {
  return 'meow' in animal;
}

function makeSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow(); // TypeScript knows it's Cat
  } else {
    animal.bark(); // TypeScript knows it's Dog
  }
}
```

---

## Quick Fire Questions

**Q: What is `any` vs `unknown`?**
```typescript
let a: any = "hello";
a.toUpperCase(); // ✅ TypeScript allows anything — type safety disabled!

let b: unknown = "hello";
b.toUpperCase(); // ❌ Error — must narrow first

if (typeof b === "string") {
  b.toUpperCase(); // ✅ Now TypeScript knows it's safe
}
// Use `unknown` instead of `any` when you're unsure of the type
```

**Q: What is `never`?**
```typescript
// Never — a type that can never occur
function throwError(message: string): never {
  throw new Error(message); // never returns
}

// Useful for exhaustive checks
function handleShape(shape: Shape) {
  switch (shape.kind) {
    case "circle": return;
    case "square": return;
    // If you add a new shape and forget to handle it:
    default:
      const _exhaustive: never = shape; // ❌ TypeScript error!
  }
}
```

**Q: What is `keyof`?**
```typescript
interface User { id: number; name: string; email: string; }
type UserKey = keyof User; // "id" | "name" | "email"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
getProperty(user, "name"); // ✅
getProperty(user, "phone"); // ❌ Error — "phone" not a key of User
```
