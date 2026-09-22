**ES6 (ECMAScript 2015)** was the biggest update the JavaScript language has ever received — it introduced classes, arrow functions, `let`/`const`, template literals, destructuring, modules, promises, and much more in a single release. Since then, ECMAScript has moved to a **yearly release cycle**, adding smaller, incremental features every year. This file is a tour of ES6 and beyond: a quick summary of features covered in depth elsewhere, plus full coverage of the features that don't have their own dedicated file.

---

### **What Is ECMAScript, and Why Did ES6 Matter?**
**ECMAScript (ES)** is the specification that JavaScript implements. Before ES6 (released in **2015**, hence also called **ES2015**), JavaScript had gone six years without a major update (the previous version, ES5, shipped in 2009). Writing non-trivial JavaScript meant relying on libraries (like jQuery or Underscore) to paper over missing language features — no real classes, no block scoping, no native modules, no promises.

ES6 changed that almost overnight:
- It gave JavaScript **block-scoped variables** (`let`/`const`) to replace the footguns of `var`.
- It added **classes** and **modules**, making large codebases far easier to structure.
- It introduced **promises**, taming callback-heavy asynchronous code.
- It added **arrow functions**, **template literals**, and **destructuring**, making everyday code shorter and more expressive.

Because of its size and impact, "ES6" is often used loosely to mean "modern JavaScript," even though the language has kept evolving every year since.

---

### **Timeline of Yearly Releases (ES2016+)**
After ES6, TC39 (the committee that governs JavaScript) moved to shipping **one release per year**, each adding a smaller, focused set of features.

| Version | Year | Notable Additions |
|---|---|---|
| **ES6 / ES2015** | 2015 | `let`/`const`, classes, arrow functions, template literals, destructuring, modules, promises, `for...of`, default/rest/spread |
| **ES2016** | 2016 | `Array.prototype.includes()`, exponentiation operator (`**`) |
| **ES2017** | 2017 | `async`/`await`, `Object.entries()`/`Object.values()`, string padding (`padStart`/`padEnd`) |
| **ES2018** | 2018 | Object rest/spread (`{...obj}`), asynchronous iteration (`for await...of`), regex improvements |
| **ES2019** | 2019 | `Array.prototype.flat()`/`flatMap()`, `Object.fromEntries()`, optional `catch` binding |
| **ES2020** | 2020 | Optional chaining (`?.`), nullish coalescing (`??`), `Promise.allSettled()`, `BigInt`, dynamic `import()` |
| **ES2021** | 2021 | `String.prototype.replaceAll()`, logical assignment operators (`&&=`, `||=`, `??=`), `Promise.any()` |
| **ES2022** | 2022 | Top-level `await`, class fields/private methods (`#field`), `Array.prototype.at()` |
| **ES2023** | 2023 | `Array.prototype.toSorted()`/`toReversed()`/`with()` (non-mutating array methods) |
| **ES2024+** | 2024+ | `Object.groupBy()`, `Promise.withResolvers()`, and continued incremental additions |

Each yearly release is small enough that it usually goes unnoticed on its own, but the cumulative effect since 2015 has been enormous.

---

### **`let` and `const` (Recap)**
ES6 introduced `let` and `const` as block-scoped alternatives to `var`, fixing hoisting and re-declaration issues that had caused bugs for years.

```javascript
let count = 1;
const MAX = 100;
```

This topic is covered in full depth — including hoisting, the temporal dead zone, and `var` vs `let` vs `const` — in **`VariablesAndDataTypes.md`**.

---

### **Template Literals (Recap)**
Template literals use backticks and `${}` to interpolate variables and expressions directly into strings, and support multi-line strings without `\n`.

```javascript
const name = "Ada";
console.log(`Hello, ${name}!`);
// Output: Hello, Ada!
```

The full details — nested templates, function calls inside interpolation, tagged templates — are covered in **`Interpolation.md`**.

---

### **Arrow Functions (Recap)**
Arrow functions provide a shorter syntax for writing functions and, unlike regular functions, do not bind their own `this`, `arguments`, or `super` — they inherit these from the enclosing scope.

```javascript
const square = (x) => x * x;
console.log(square(5));
// Output: 25
```

Arrow function syntax variations, implicit returns, and `this`-binding behavior are covered in full in **`Functions.md`**.

---

### **Default Parameters**
Function parameters can be given default values, which are used when the argument is `undefined` or omitted entirely.

```javascript
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

console.log(greet());
// Output: Hello, Guest!
console.log(greet("Sam"));
// Output: Hello, Sam!
console.log(greet("Sam", "Welcome"));
// Output: Welcome, Sam!
```

Default values can also reference earlier parameters:
```javascript
function createUser(name, role = "member", label = `${name} (${role})`) {
  return label;
}

console.log(createUser("Priya", "admin"));
// Output: Priya (admin)
```

---

### **Rest Parameters**
The **rest operator** (`...`) collects any remaining arguments into a real array, replacing the old, awkward `arguments` object.

```javascript
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3, 4));
// Output: 10
```

Rest can also be combined with named parameters, as long as it comes last:
```javascript
function logScores(student, ...scores) {
  console.log(student, scores);
}

logScores("Ravi", 90, 85, 95);
// Output: Ravi [ 90, 85, 95 ]
```

---

### **Spread Operator**
The **spread operator** looks identical to rest (`...`) but does the opposite job — it expands an iterable (array, string, or object) into individual elements.

```javascript
// Spreading arrays
const nums1 = [1, 2, 3];
const nums2 = [4, 5, 6];
const combined = [...nums1, ...nums2];
console.log(combined);
// Output: [ 1, 2, 3, 4, 5, 6 ]

// Spreading into function arguments
console.log(Math.max(...nums1));
// Output: 3

// Spreading objects (shallow clone / merge)
const base = { role: "user" };
const admin = { ...base, role: "admin", canDelete: true };
console.log(admin);
// Output: { role: 'admin', canDelete: true }
```

#### **Rest vs Spread**
| | Rest (`...`) | Spread (`...`) |
|---|---|---|
| **Purpose** | Collects multiple values **into** an array/object | Expands an array/object **out** into individual values |
| **Where used** | Function parameters, destructuring | Function calls, array/object literals |
| **Example** | `function f(...args) {}` | `f(...myArray)` |

---

### **Destructuring Assignment (Recap)**
Destructuring lets you unpack values from arrays or properties from objects into distinct variables in a single expression.

```javascript
// Array destructuring
const [first, second] = ["Alice", "Bob"];

// Object destructuring
const { name, age } = { name: "Sara", age: 28 };

console.log(first, second, name, age);
// Output: Alice Bob Sara 28
```

Array destructuring is covered with skipping elements, swapping, and defaults in **`Arrays.md`**; object destructuring — including renaming, nested destructuring, and defaults — is covered in **`Objects.md`**.

---

### **Classes (Brief)**
ES6 `class` syntax provides a cleaner, more familiar way to create constructor functions and set up prototype-based inheritance.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound.`;
  }
}

class Dog extends Animal {
  speak() {
    return `${this.name} barks.`;
  }
}

console.log(new Dog("Rex").speak());
// Output: Rex barks.
```

`class` is syntactic sugar over JavaScript's existing prototype chain — it does not introduce a new inheritance model. The full mechanics, including `super`, static methods, private fields (`#field`), and how classes relate to prototypes, are covered in **`PrototypesAndInheritance.md`**.

---

### **Modules (Brief)**
ES6 introduced a native module system using `import` and `export`, replacing ad-hoc patterns like IIFEs or third-party module loaders.

```javascript
// math.js
export const add = (a, b) => a + b;

// app.js
import { add } from "./math.js";
console.log(add(2, 3));
// Output: 5
```

Modules — including default exports, re-exporting, CommonJS, and dynamic `import()` — are covered in full in **`Modules.md`**.

---

### **`Symbol`**
`Symbol` is a new **primitive type** introduced in ES6 that creates a guaranteed-unique value, most often used as a non-colliding object property key.

```javascript
const id = Symbol("id");
const user = {
  name: "Tom",
  [id]: 12345,
};

console.log(user[id]);
// Output: 12345
console.log(Object.keys(user));
// Output: [ 'name' ]  (symbol keys are hidden from normal enumeration)
```

Symbols are commonly used internally by the language too — for example, `Symbol.iterator` defines how an object behaves with `for...of`.

---

### **`Map` and `Set`**
ES6 added two new collection types that address long-standing limitations of plain objects and arrays.

#### **`Map`**
A `Map` stores key-value pairs where **keys can be any type** (not just strings), and it preserves insertion order.

```javascript
const scores = new Map();
scores.set("Alice", 90);
scores.set("Bob", 85);

console.log(scores.get("Alice"));
// Output: 90
console.log(scores.size);
// Output: 2

for (const [player, score] of scores) {
  console.log(`${player}: ${score}`);
}
// Output:
// Alice: 90
// Bob: 85
```

#### **`Set`**
A `Set` stores a collection of **unique values** — it automatically discards duplicates.

```javascript
const unique = new Set([1, 2, 2, 3, 3, 3]);
console.log([...unique]);
// Output: [ 1, 2, 3 ]

unique.add(4);
console.log(unique.has(2));
// Output: true
```

#### **`WeakMap` and `WeakSet` (Briefly)**
`WeakMap` and `WeakSet` are variants that only accept **objects** as keys/values and hold **weak references** to them — meaning entries are automatically garbage-collected once nothing else references the object. They're not iterable and have no `size` property, which makes them useful for attaching private, memory-safe metadata to objects (e.g., caching data tied to a DOM node) without causing memory leaks.

#### **Map vs Object, Set vs Array**
| | `Map` | Plain Object |
|---|---|---|
| **Key types** | Any value | Strings/Symbols only |
| **Order** | Insertion order guaranteed | Mostly insertion order (with caveats) |
| **Size** | `.size` property | Manual (`Object.keys(obj).length`) |
| **Iterable** | Yes, directly | No (needs `Object.entries()`, etc.) |

| | `Set` | Array |
|---|---|---|
| **Duplicates** | Not allowed | Allowed |
| **Lookup speed** | O(1) via `.has()` | O(n) via `.includes()` |
| **Order** | Insertion order | Index order |

---

### **`Promise` (Brief)**
A `Promise` represents a value that will be available now, later, or never — it's the foundation of modern asynchronous JavaScript, replacing deeply nested callbacks.

```javascript
const fetchData = new Promise((resolve, reject) => {
  setTimeout(() => resolve("data loaded"), 1000);
});

fetchData.then((result) => console.log(result));
// Output (after ~1s): data loaded
```

Promises, chaining, `Promise.all`/`allSettled`/`race`, and `async`/`await` are covered in depth in **`AsynchronousJavaScript.md`**.

---

### **The `for...of` Loop**
`for...of` iterates over the **values** of any iterable (arrays, strings, Maps, Sets, etc.), unlike `for...in`, which iterates over **keys/indices**.

```javascript
const colors = ["red", "green", "blue"];

for (const color of colors) {
  console.log(color);
}
// Output:
// red
// green
// blue

for (const char of "hi") {
  console.log(char);
}
// Output:
// h
// i
```

#### **`for...of` vs `for...in`**
| | `for...of` | `for...in` |
|---|---|---|
| **Iterates over** | Values | Keys (property names/indices) |
| **Works on** | Iterables (Array, String, Map, Set, etc.) | Any enumerable object |
| **Common use** | Looping over array/collection values | Looping over object property names |

---

### **Optional Chaining (`?.`) and Nullish Coalescing (`??`) (Recap)**
Introduced in **ES2020**, these operators make working with potentially missing data much safer and shorter.

**Optional chaining** short-circuits to `undefined` instead of throwing when accessing a property on `null`/`undefined`:
```javascript
const user = { profile: null };
console.log(user.profile?.bio);
// Output: undefined  (no error, even though profile is null)
```

**Nullish coalescing** provides a fallback only when the left side is `null` or `undefined` — unlike `||`, it does not treat `0`, `""`, or `false` as missing:
```javascript
const count = 0;
console.log(count || 10); // Output: 10 (wrong! 0 is falsy)
console.log(count ?? 10); // Output: 0  (correct! 0 is not nullish)
```

They're frequently combined:
```javascript
const city = user.profile?.address?.city ?? "Unknown";
console.log(city);
// Output: Unknown
```

---

### **Quick-Reference Table**
| Feature | ES Version | One-Line Description |
|---|---|---|
| `let` / `const` | ES6 | Block-scoped variable declarations |
| Template literals | ES6 | String interpolation with backticks and `${}` |
| Arrow functions | ES6 | Shorter function syntax with lexical `this` |
| Default parameters | ES6 | Fallback values for missing function arguments |
| Rest / spread | ES6 | Collect (`...args`) or expand (`...arr`) values |
| Destructuring | ES6 | Unpack arrays/objects into variables |
| Classes | ES6 | Syntactic sugar over prototype-based inheritance |
| Modules (`import`/`export`) | ES6 | Native module system |
| `Promise` | ES6 | Represents an eventual async result |
| `Symbol` | ES6 | Unique, collision-free primitive value |
| `Map` / `Set` | ES6 | Keyed collection / unique-value collection |
| `for...of` | ES6 | Loop over iterable values |
| `Array.includes()` | ES2016 | Check if an array contains a value |
| `async`/`await` | ES2017 | Synchronous-looking syntax for promises |
| Object rest/spread | ES2018 | `{...obj}` for objects |
| `Array.flat()`/`flatMap()` | ES2019 | Flatten nested arrays |
| Optional chaining `?.` | ES2020 | Safe property access on possibly-null values |
| Nullish coalescing `??` | ES2020 | Fallback only for `null`/`undefined` |
| Dynamic `import()` | ES2020 | Load modules on demand |
| `BigInt` | ES2020 | Arbitrary-precision integers |
| Logical assignment (`&&=`, `||=`, `??=`) | ES2021 | Combined logical + assignment operators |
| Private class fields (`#field`) | ES2022 | True encapsulation in classes |
| Top-level `await` | ES2022 | `await` outside an `async` function in modules |

---

### **Best Practices**
- Prefer `const` by default, `let` when reassignment is needed, and avoid `var` in new code.
- Use destructuring and default parameters to make function signatures self-documenting.
- Reach for `Map`/`Set` instead of plain objects/arrays when you need non-string keys, guaranteed uniqueness, or frequent `.size`/`.has()` checks.
- Use `?.` and `??` together to safely read deeply nested, possibly-missing data instead of long chains of `&&` checks.
- Check compatibility (caniuse.com or your build target) before relying on very recent ES2022+ features in production code that must support older environments.
- Don't chase every new proposal immediately — TC39 features go through stages, and only "Stage 4" (finished) features are safe to rely on without a transpiler.

---

### **Interview Questions**

**Q1. What is ECMAScript, and how is it related to ES6?**
ECMAScript is the specification JavaScript implements. ES6 (also called ES2015) was the sixth edition of that specification and the largest single update the language has received, introducing `let`/`const`, classes, arrow functions, promises, modules, and more.

**Q2. Why did JavaScript move to yearly ES releases after ES6?**
Shipping one giant release every several years made it hard to plan and slow to deliver new features. TC39 switched to smaller, yearly releases (ES2016, ES2017, ...) so individual features can ship as soon as they're ready, without waiting on unrelated ones.

**Q3. What's the difference between rest and spread, since they use the same `...` syntax?**
Rest **collects** multiple values into a single array or object (used in function parameters or destructuring), while spread **expands** an array/object into individual values (used in function calls or literals). The direction — gathering vs. spreading out — tells them apart.
```javascript
function f(...args) {}   // rest: gathers arguments into an array
f(...[1, 2, 3]);         // spread: expands the array into arguments
```

**Q4. How is `Map` different from a plain object?**
`Map` allows keys of any type (objects, functions, etc.), preserves insertion order reliably, has a built-in `.size`, and is directly iterable. Plain objects only support string/Symbol keys and require helper methods like `Object.keys()` to iterate.

**Q5. What problem does `Set` solve?**
`Set` stores only unique values and automatically discards duplicates, with O(1) membership checks via `.has()`. It's commonly used to deduplicate arrays: `[...new Set(array)]`.

**Q6. What is the difference between `WeakMap`/`WeakSet` and their non-weak counterparts?**
`WeakMap`/`WeakSet` only accept objects as keys and hold weak references, so entries can be garbage-collected once nothing else references the key object. They aren't iterable and have no `.size`, making them suitable for attaching metadata to objects without risking memory leaks.

**Q7. What's the difference between `for...of` and `for...in`?**
`for...of` iterates over the values of an iterable (arrays, strings, Maps, Sets); `for...in` iterates over the enumerable property keys of an object (and, on arrays, its indices as strings). `for...of` is generally preferred for arrays.

**Q8. How does `??` differ from `||`?**
`||` returns the right-hand value whenever the left is any falsy value (`0`, `""`, `false`, `null`, `undefined`, `NaN`). `??` only falls back when the left side is specifically `null` or `undefined`, making it safer for values where `0` or `""` are valid.

**Q9. What does optional chaining (`?.`) do?**
It short-circuits and returns `undefined` instead of throwing a `TypeError` when accessing a property, calling a method, or indexing on something that turns out to be `null` or `undefined`, e.g. `obj?.a?.b?.()`.

**Q10. What is a `Symbol` used for?**
`Symbol` creates a unique, immutable value typically used as an object property key to avoid naming collisions, including with the language's own internal behaviors (like `Symbol.iterator`, which defines how an object is looped over with `for...of`).

**Q11. Are ES6 classes "real" classes like in Java or C++?**
No — they are syntactic sugar over JavaScript's existing prototype-based inheritance model. Under the hood, methods defined in a class still live on the constructor's `.prototype`, and instances still resolve properties via the prototype chain.

**Q12. What's the difference between CommonJS and ES modules, at a glance?**
CommonJS (`require`/`module.exports`) is Node.js's original, synchronous module system. ES modules (`import`/`export`) are the native, standardized system introduced in ES6, supporting static analysis and asynchronous loading — used natively in browsers and modern Node.js. Full details are in `Modules.md`.
