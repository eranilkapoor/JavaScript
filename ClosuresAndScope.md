**Closures and Scope** are two of the most fundamental — and most misunderstood — concepts in JavaScript. Scope determines *where* a variable is accessible, while a closure is the mechanism that lets a function "remember" the variables from the scope it was created in, even after that outer scope has finished executing. Understanding these deeply is essential for writing correct, bug-free JavaScript and is one of the most common topics asked about in interviews.

---

### **What is Scope?**
**Scope** defines the accessibility (visibility) of variables in different parts of your code. JavaScript resolves variable names by looking at the scope in which the code is written, not where it is called from (this is called **lexical scoping**).

There are three main types of scope in JavaScript:

1. **Global Scope**: Variables declared outside any function or block. They are accessible from anywhere in the program.
2. **Function Scope**: Variables declared inside a function using `var`, `let`, or `const` are only accessible within that function.
3. **Block Scope**: Variables declared inside a pair of curly braces `{}` (e.g., `if`, `for`, `while`) using `let` or `const` are only accessible within that block.

---

### **Global Scope Example**
```javascript
const appName = "MyApp"; // Global scope

function greet() {
  console.log(`Welcome to ${appName}`); // Accessible here
}

greet();
// Output: Welcome to MyApp
console.log(appName);
// Output: MyApp
```

---

### **Function Scope Example**
```javascript
function calculate() {
  var result = 42; // Function-scoped
  console.log(result);
  // Output: 42
}

calculate();
console.log(typeof result);
// Output: undefined (result is not accessible outside the function)
```

---

### **Block Scope: `let`/`const` vs `var`**
This is one of the most important distinctions in modern JavaScript. `var` is **function-scoped** (or globally scoped if declared outside a function), while `let` and `const` are **block-scoped**.

```javascript
if (true) {
  var oldWay = "I leak out of the block";
  let newWay = "I stay inside the block";
  const alsoNewWay = "I stay inside the block too";
}

console.log(oldWay);
// Output: I leak out of the block

console.log(newWay);
// Output: ReferenceError: newWay is not defined
```

#### **Key Differences**

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| **Scope** | Function-scoped | Block-scoped | Block-scoped |
| **Re-declaration** | Allowed | Not allowed in same scope | Not allowed in same scope |
| **Re-assignment** | Allowed | Allowed | Not allowed |
| **Hoisting** | Hoisted, initialized as `undefined` | Hoisted, but in the "temporal dead zone" | Hoisted, but in the "temporal dead zone" |
| **Attached to `window`/global object** | Yes (in browsers) | No | No |

---

### **Lexical Scoping and the Scope Chain**
**Lexical scoping** means that a function's access to variables is determined by *where the function is physically written* in the source code, not by where or how it is called. When JavaScript tries to resolve a variable, it looks in the current scope first, and if it doesn't find it, it walks *up* through each enclosing (parent) scope — this chain of nested scopes is called the **scope chain**.

```javascript
const outerVar = "I'm outside";

function outerFunction() {
  const innerVar = "I'm inside outerFunction";

  function innerFunction() {
    const innermostVar = "I'm inside innerFunction";
    console.log(outerVar);   // Found in global scope
    console.log(innerVar);   // Found in outerFunction's scope
    console.log(innermostVar); // Found in its own scope
  }

  innerFunction();
}

outerFunction();
// Output:
// I'm outside
// I'm inside outerFunction
// I'm inside innerFunction
```

The scope chain only works **outward**. `outerFunction` cannot access `innermostVar`, because a function cannot see into scopes nested below it — only into the scopes it is nested within.

---

### **What is a Closure?**
A **closure** is formed when an inner function retains access to the variables of its outer (enclosing) function's scope, even after the outer function has finished executing. In other words, the inner function "closes over" the variables it needs, preserving them in memory instead of letting them be garbage collected.

#### **Step-by-Step Example**
```javascript
function createGreeter(greeting) {
  // `greeting` lives in createGreeter's scope
  return function (name) {
    // This inner function forms a closure over `greeting`
    console.log(`${greeting}, ${name}!`);
  };
}

const sayHello = createGreeter("Hello");
const sayHi = createGreeter("Hi");

sayHello("Alice");
// Output: Hello, Alice!

sayHi("Bob");
// Output: Hi, Bob!
```

**What's happening here, step by step:**
1. `createGreeter("Hello")` is called and starts executing. A new scope is created with `greeting = "Hello"`.
2. `createGreeter` returns an anonymous function that references `greeting`.
3. Normally, `greeting` would be destroyed once `createGreeter` finishes running — but because the returned function references it, JavaScript keeps `greeting` alive in memory.
4. `sayHello` now holds a reference to that inner function, along with its own private "backpack" containing `greeting = "Hello"`.
5. Calling `sayHello("Alice")` uses the remembered `greeting` value, even though `createGreeter` already returned.
6. `sayHi` has its own independent closure with `greeting = "Hi"` — each call to `createGreeter` creates a fresh, isolated scope.

---

### **Practical Use Cases for Closures**

#### **1. Counters and Private State**
Closures let you emulate private variables — data that can only be modified through controlled functions, similar to encapsulation in OOP.

```javascript
function createCounter() {
  let count = 0; // private variable, not accessible from outside

  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
console.log(counter.increment()); // Output: 1
console.log(counter.increment()); // Output: 2
console.log(counter.decrement()); // Output: 1
console.log(counter.count);       // Output: undefined (truly private)
```

#### **2. Memoization**
Closures can cache the results of expensive function calls so repeated calls with the same input return instantly.

```javascript
function memoize(fn) {
  const cache = {}; // remembered across calls via closure

  return function (...args) {
    const key = JSON.stringify(args);
    if (cache[key] !== undefined) {
      console.log("Returning from cache");
      return cache[key];
    }
    const result = fn(...args);
    cache[key] = result;
    return result;
  };
}

const slowSquare = (n) => {
  for (let i = 0; i < 1e6; i++) {} // simulate expensive work
  return n * n;
};

const fastSquare = memoize(slowSquare);
console.log(fastSquare(5)); // Output: 25 (computed)
console.log(fastSquare(5)); // Output: Returning from cache \n 25
```

#### **3. Function Factories**
A **function factory** is a function that generates other functions, each pre-configured with different behavior via closures.

```javascript
function multiplyBy(factor) {
  return function (number) {
    return number * factor;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);

console.log(double(5)); // Output: 10
console.log(triple(5)); // Output: 15
```

#### **4. Partial Application**
Closures can "lock in" some arguments of a function ahead of time, producing a new function that expects only the remaining arguments.

```javascript
function partial(fn, ...presetArgs) {
  return function (...remainingArgs) {
    return fn(...presetArgs, ...remainingArgs);
  };
}

function add(a, b, c) {
  return a + b + c;
}

const addFiveAndTen = partial(add, 5, 10);
console.log(addFiveAndTen(20));
// Output: 35
```

---

### **Classic Pitfall: Closures Inside Loops**
A very common interview question and real-world bug involves closures capturing the wrong variable inside a loop.

#### **The Problem (with `var`)**
```javascript
for (var i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
// Output (after 100ms):
// 4
// 4
// 4
```

**Why this happens**: `var` is function-scoped, not block-scoped. There is only **one** `i` shared across all iterations of the loop. By the time the `setTimeout` callbacks actually run (after the loop has already finished), `i` has already been incremented to `4`. All three closures reference the *same* `i` variable, which now holds its final value.

#### **The Fix (with `let`)**
```javascript
for (let i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
// Output (after 100ms):
// 1
// 2
// 3
```

**Why this works**: `let` is block-scoped, so JavaScript creates a **brand-new binding of `i` for every single iteration** of the loop. Each `setTimeout` callback closes over its own independent copy of `i`, capturing the value it had during that specific iteration.

#### **Fixing it with `var` (for comparison)**
If you must use `var`, you can fix the bug manually by creating a new scope with an IIFE (Immediately Invoked Function Expression):

```javascript
for (var i = 1; i <= 3; i++) {
  (function (capturedI) {
    setTimeout(function () {
      console.log(capturedI);
    }, 100);
  })(i);
}
// Output (after 100ms):
// 1
// 2
// 3
```

---

### **Getters and Setters via Closures**
Closures can be combined with `Object.defineProperty` or factory functions to implement controlled access to a value, similar to getters/setters in other languages.

```javascript
function createTemperature() {
  let celsius = 0;

  return {
    getCelsius() {
      return celsius;
    },
    setCelsius(value) {
      if (typeof value !== "number") {
        throw new TypeError("Temperature must be a number");
      }
      celsius = value;
    },
    getFahrenheit() {
      return celsius * 1.8 + 32;
    },
  };
}

const temp = createTemperature();
temp.setCelsius(25);
console.log(temp.getCelsius());    // Output: 25
console.log(temp.getFahrenheit()); // Output: 77
```

This can also be written using native `get`/`set` syntax, where the closure still protects the private variable:

```javascript
function createAccount(initialBalance) {
  let balance = initialBalance;

  return {
    get balance() {
      return balance;
    },
    set balance(amount) {
      if (amount < 0) {
        console.log("Balance cannot be negative");
        return;
      }
      balance = amount;
    },
  };
}

const account = createAccount(100);
console.log(account.balance); // Output: 100
account.balance = 250;
console.log(account.balance); // Output: 250
account.balance = -50;
// Output: Balance cannot be negative
console.log(account.balance); // Output: 250
```

---

### **Best Practices**
- Prefer `let` and `const` over `var` to avoid scope-related bugs and get block scoping by default.
- Use closures deliberately for encapsulation (private state) rather than relying on global variables.
- Be mindful of memory: closures keep referenced variables alive, so avoid closing over large objects or DOM nodes you no longer need.
- When using closures for memoization or caching, consider cache eviction strategies to avoid unbounded memory growth.
- Prefer `let` inside loops that create closures (like `setTimeout` or event listener callbacks) to avoid the classic shared-variable bug.
- Keep closures small and focused — a closure that captures too many outer variables is harder to reason about and debug.

---

### **Interview Questions**

**Q1. What is a closure in JavaScript?**
A closure is a function that retains access to variables from its outer (enclosing) lexical scope, even after that outer function has returned. It happens automatically whenever a function is defined inside another function and referenced outside of it.

**Q2. What is the difference between scope and closure?**
Scope defines *where* a variable is accessible in the code (global, function, or block). A closure is the *mechanism* by which an inner function preserves access to its outer scope's variables even after the outer function has finished executing. Closures are built on top of lexical scoping.

**Q3. Why does the classic `var` in a loop with `setTimeout` print the same final value for every callback?**
Because `var` is function-scoped rather than block-scoped, there is only one shared variable across all loop iterations. By the time the asynchronous callbacks run, the loop has already completed and the variable holds its final value.
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 3 3 3
```

**Q4. How does `let` fix the closure-in-loop problem?**
`let` is block-scoped, so the JavaScript engine creates a new binding of the loop variable for each iteration. Each closure formed inside the loop body captures its own distinct copy of the variable instead of a single shared one.

**Q5. What is the difference between `var`, `let`, and `const`?**
`var` is function-scoped and hoisted with an `undefined` initial value, allowing re-declaration. `let` and `const` are block-scoped and live in the "temporal dead zone" until their declaration line executes. `let` allows reassignment, while `const` does not allow the binding to be reassigned (though objects/arrays assigned to a `const` can still be mutated).

**Q6. How can closures be used to create private variables in JavaScript?**
By declaring a variable inside an outer function and returning inner functions that reference it, the variable becomes inaccessible from outside except through the returned functions — effectively emulating private state.
```javascript
function counter() {
  let count = 0;
  return () => ++count;
}
const inc = counter();
inc(); inc();
console.log(inc()); // Output: 3
```

**Q7. What is the scope chain?**
The scope chain is the ordered list of scopes the JavaScript engine searches through when resolving a variable — starting with the current (innermost) scope and walking outward through each enclosing scope until it reaches the global scope. If a variable isn't found anywhere in the chain, a `ReferenceError` is thrown.

**Q8. Can closures cause memory leaks?**
Yes. Because a closure keeps a reference to its outer scope's variables alive, if a closure is kept around (e.g., attached to a long-lived event listener or stored globally) and it references large objects or DOM elements, those objects cannot be garbage collected even if they're no longer needed elsewhere.

**Q9. What is the difference between a function factory and partial application, and how do closures enable both?**
A function factory returns a new function configured with values captured via closure (e.g., `multiplyBy(2)` returns a "double" function). Partial application is a specific case where some arguments of an existing function are pre-filled via closure, returning a new function that only needs the remaining arguments.

**Q10. What is the temporal dead zone (TDZ)?**
The TDZ is the period between entering a scope and the point where a `let` or `const` variable is actually declared. Accessing the variable during this period throws a `ReferenceError`, even though the variable is technically hoisted to the top of the scope.
```javascript
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 5;
```

**Q11. How would you implement a simple memoized function using closures?**
Wrap the target function in another function that maintains a cache object via closure. On each call, check whether the arguments already have a cached result; if so, return it, otherwise compute, store, and return the new result.

**Q12. Does every function in JavaScript create a closure?**
Technically, every function forms a closure over its lexical scope, whether or not it uses any outer variables. In practice, we only call something "a closure" in a meaningful sense when the inner function actually references and relies on variables from an outer scope after that scope would otherwise have ended.
