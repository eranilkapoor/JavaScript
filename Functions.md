**Functions** are reusable blocks of code designed to perform a specific task. JavaScript treats functions as **first-class citizens**, meaning they can be stored in variables, passed as arguments, and returned from other functions — a feature that underpins much of the language's flexibility, including callbacks and higher-order functions.

---

### **Function Declarations vs Function Expressions vs Arrow Functions**

#### **1. Function Declaration**
```javascript
function add(a, b) {
  return a + b;
}
console.log(add(2, 3)); // Output: 5
```
- **Hoisted**: The entire function (name and body) is hoisted, so it can be called before its definition in the code.
- Has its own `this`, determined by how it's **called** (not where it's defined).
- Has access to the `arguments` object.

```javascript
console.log(greet()); // Output: Hi! (works even though called before definition)
function greet() {
  return "Hi!";
}
```

#### **2. Function Expression**
```javascript
const multiply = function (a, b) {
  return a * b;
};
console.log(multiply(2, 3)); // Output: 6
```
- **Not hoisted** the same way — the variable (`multiply`) is hoisted, but its assignment happens only when that line executes, so calling it earlier throws a `TypeError` (or `ReferenceError` with `let`/`const`).
- Also has its own `this` and `arguments`, just like a declaration.

```javascript
// console.log(subtract(5, 2)); // TypeError: subtract is not a function
const subtract = function (a, b) {
  return a - b;
};
```

#### **3. Arrow Function (ES6)**
```javascript
const divide = (a, b) => a / b;
console.log(divide(10, 2)); // Output: 5
```
- **Not hoisted** in a usable way (same as function expressions, since they're typically assigned to `const`/`let`).
- **No own `this`** — arrow functions inherit `this` **lexically** from their surrounding scope. This is the single biggest behavioral difference.
- **No `arguments` object** — you must use rest parameters (`...args`) instead.
- Cannot be used as a constructor (`new ArrowFn()` throws an error).

```javascript
const obj = {
  name: "Timer",
  regularFn: function () {
    console.log(this.name); // Output: Timer  (this = obj, the caller)
  },
  arrowFn: () => {
    console.log(this.name); // Output: undefined (this = outer/lexical scope, not obj)
  },
};
obj.regularFn();
obj.arrowFn();
```

#### **Comparison Table**

| Feature | Function Declaration | Function Expression | Arrow Function |
|---|---|---|---|
| Hoisting | Fully hoisted (usable before definition) | Not usable before assignment | Not usable before assignment |
| `this` binding | Dynamic (depends on caller) | Dynamic (depends on caller) | Lexical (inherited from enclosing scope) |
| `arguments` object | Available | Available | Not available (use rest params) |
| Can be used as constructor (`new`) | Yes | Yes (if not arrow) | No |
| Syntax verbosity | Verbose | Verbose | Concise |

---

### **Default Parameters**
Allows a parameter to fall back to a default value if the caller doesn't provide one (or passes `undefined`).
```javascript
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}
console.log(greet());        // Output: Hello, Guest!
console.log(greet("Alice")); // Output: Hello, Alice!
console.log(greet(undefined)); // Output: Hello, Guest!  (undefined triggers default)
console.log(greet(null));      // Output: Hello, null!   (null does NOT trigger default)
```

---

### **Rest Parameters**
Collects all remaining arguments into a real array, unlike the older `arguments` object which is only array-like.
```javascript
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}
console.log(sum(1, 2, 3, 4)); // Output: 10

function logFirst(first, ...rest) {
  console.log(first, rest);
}
logFirst(1, 2, 3, 4); // Output: 1 [ 2, 3, 4 ]
```

---

### **IIFE (Immediately Invoked Function Expression)**
A function that runs as soon as it's defined, wrapped in parentheses to make it an expression rather than a declaration. Historically used to avoid polluting the global scope before `let`/`const`/modules existed.
```javascript
(function () {
  const secret = "hidden";
  console.log("IIFE ran!", secret);
})();
// Output: IIFE ran! hidden

// Arrow function IIFE
(() => {
  console.log("Arrow IIFE ran!");
})();
// Output: Arrow IIFE ran!
```
**Why use it?**
- Creates a private scope so internal variables (`secret`) don't leak into the global scope.
- Useful for one-time setup/initialization code, and still common in module patterns and bundler output.

---

### **First-Class Functions**
Because functions are values in JavaScript, they can be:

1. **Assigned to variables**:
   ```javascript
   const sayHi = function () { return "Hi!"; };
   ```
2. **Passed as arguments** (see Callback Functions below).
3. **Returned from other functions**:
   ```javascript
   function createMultiplier(factor) {
     return function (number) {
       return number * factor;
     };
   }
   const double = createMultiplier(2);
   console.log(double(5)); // Output: 10
   ```
4. **Stored in data structures** (arrays, objects):
   ```javascript
   const operations = {
     add: (a, b) => a + b,
     subtract: (a, b) => a - b,
   };
   console.log(operations.add(4, 2)); // Output: 6
   ```

---

### **Higher-Order Functions**
A **higher-order function** either takes one or more functions as arguments, returns a function, or both. `createMultiplier` above is one example; array methods like `map`, `filter`, and `reduce` (covered in `Arrays.md`) are the most common examples in everyday code.

```javascript
function applyOperation(a, b, operation) {
  return operation(a, b);
}

const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

console.log(applyOperation(4, 5, add));      // Output: 9
console.log(applyOperation(4, 5, multiply)); // Output: 20
```

---

### **Callback Functions**
A **callback** is a function passed into another function to be executed later — commonly used for event handling, array iteration, and asynchronous operations (covered in depth in `AsynchronousJavaScript.md`).
```javascript
function processUserInput(callback) {
  const name = "Alice";
  callback(name);
}

processUserInput((name) => {
  console.log(`Processed: ${name}`);
});
// Output: Processed: Alice

// A very common real-world callback usage
[1, 2, 3].forEach((num) => console.log(num * 2));
// Output: 2
//         4
//         6

setTimeout(() => console.log("Runs after 1 second"), 1000);
```

---

### **Pass by Value vs Pass by Reference**
When arguments are passed to a function, **primitives are passed by value** (a copy), while **objects and arrays are passed by reference** (a copy of the reference, pointing to the same underlying data).

#### **Primitives (Pass by Value)**
```javascript
function increment(num) {
  num = num + 1;
  return num;
}

let value = 10;
increment(value);
console.log(value); // Output: 10 (unchanged — the function worked on a copy)
```

#### **Objects/Arrays (Pass by Reference)**
```javascript
function addProperty(obj) {
  obj.newProp = "added";
}

let person = { name: "Alice" };
addProperty(person);
console.log(person); // Output: { name: 'Alice', newProp: 'added' } (mutated!)
```

However, **reassigning** the parameter inside the function does **not** affect the original reference, because the reference itself was copied:
```javascript
function reassign(obj) {
  obj = { name: "New Object" }; // only reassigns the local copy of the reference
}

let original = { name: "Original" };
reassign(original);
console.log(original); // Output: { name: 'Original' } (unchanged)
```

| Type | Passed as | Mutating properties/elements affects original? | Reassigning the parameter affects original? |
|---|---|---|---|
| Primitive (Number, String, Boolean, etc.) | Value (copy) | N/A | No |
| Object / Array | Reference (copy of the reference) | Yes | No |

---

### **Best Practices**
- Prefer arrow functions for short callbacks and cases where you want lexical `this` (e.g., inside class methods or event handlers).
- Use regular functions (declarations/expressions) when you need `this` to be dynamic, `arguments`, or when using the function as a constructor.
- Give functions clear, single-purpose responsibilities (the Single Responsibility Principle).
- Use default parameters instead of manual `if (x === undefined)` checks.
- Prefer rest parameters over the legacy `arguments` object — it's a real array with all array methods available.
- Avoid mutating objects/arrays passed as parameters unless that side effect is intentional and documented.

---

### **Interview Questions**

**Q1. What's the difference between a function declaration and a function expression?**
A function declaration (`function foo() {}`) is fully hoisted, including its body, so it can be called before it appears in the code. A function expression (`const foo = function() {}`) is only hoisted as an uninitialized variable, so calling it before the assignment throws an error.

**Q2. How does `this` behave differently in arrow functions vs regular functions?**
Regular functions have a dynamic `this` determined by how the function is called (the receiver). Arrow functions have no `this` of their own — they inherit `this` lexically from the enclosing scope at the time they're defined, which never changes regardless of how they're invoked.

**Q3. Why can't arrow functions be used as constructors?**
Because they don't have their own `this` binding or a `prototype` property, both of which the `new` operator relies on to create and link a new instance. Calling `new` on an arrow function throws a `TypeError`.

**Q4. What is an IIFE, and why is it used?**
An Immediately Invoked Function Expression is a function that executes right after it's defined, e.g., `(function() { ... })();`. It creates an isolated scope, historically used to avoid leaking variables into the global scope before block scoping and modules existed.

**Q5. What are rest parameters, and how do they differ from the `arguments` object?**
Rest parameters (`...args`) collect the remaining arguments into a real `Array`, with all array methods available. The `arguments` object is only array-like (has a `length` and index access, but not `map`/`filter`/etc.) and isn't available in arrow functions.

**Q6. What makes a function "higher-order"?**
A higher-order function either accepts one or more functions as arguments, returns a new function, or both. Examples include `Array.prototype.map`, `Array.prototype.filter`, and custom utilities like a `compose` or `debounce` function.

**Q7. Are objects passed by reference in JavaScript?**
Not exactly — JavaScript is always pass-by-value, but for objects, the "value" being copied is the **reference** (memory address) to the object. That's why mutating an object inside a function affects the original, but reassigning the parameter to a brand-new object does not.

**Q8. What happens when you pass a primitive to a function and modify it inside?**
The function receives a copy of the value, so any reassignment inside the function only affects the local copy — the original variable outside remains unchanged.

**Q9. What is a callback function? Give an example.**
A callback is a function passed as an argument to another function, to be invoked later — either synchronously (like in `Array.prototype.forEach`) or asynchronously (like in `setTimeout` or an event listener). Example: `button.addEventListener("click", () => console.log("clicked"))`.

**Q10. What's the default value behavior with `undefined` vs `null` in default parameters?**
Default parameters only kick in when the argument is `undefined` (or omitted entirely). Passing `null` explicitly does **not** trigger the default — `null` is treated as a valid, intentional value.

**Q11. Can you change what `this` refers to in a regular function? What about an arrow function?**
Yes, for regular functions, using `call()`, `apply()`, `bind()`, or by changing how/where the function is invoked. Arrow functions cannot have their `this` changed by any of these methods — it's permanently locked to the lexical scope where the arrow function was defined.

**Q12. Why might closures over `var` in a loop combined with function references cause bugs, and how do first-class functions relate?**
Because functions are first-class values captured by reference to their surrounding scope, callbacks created inside a `var` loop all close over the same function-scoped variable, so by the time they run they all see its final value. Using `let` (block-scoped per iteration) or generating a new closure per iteration fixes it — this is explored further in `ClosuresAndScope.md`.
