**Error Handling** in JavaScript is how a program detects, responds to, and recovers from unexpected problems — invalid input, failed network requests, or bugs in logic — instead of crashing outright. Writing robust applications requires knowing not just how to catch errors, but how to classify them, create meaningful custom errors, and handle failures correctly in both synchronous and asynchronous code.

---

### **`try`/`catch`/`finally` Syntax**
The `try` block contains code that might throw an error. If an error occurs, control immediately jumps to the `catch` block. The `finally` block runs **always**, whether or not an error occurred, and is typically used for cleanup.

```javascript
try {
  console.log("Step 1: Trying...");
  JSON.parse("{ invalid json }"); // Throws a SyntaxError
  console.log("Step 2: This never runs");
} catch (error) {
  console.log("Caught an error:", error.message);
} finally {
  console.log("Cleanup: this always runs");
}

// Output:
// Step 1: Trying...
// Caught an error: Unexpected token i in JSON at position 2
// Cleanup: this always runs
```

#### **Execution Order**
1. Code in `try` runs until an error is thrown (or it completes successfully).
2. If an error is thrown, execution jumps immediately to `catch`, skipping the rest of `try`.
3. `finally` always executes last, regardless of whether an error was thrown, caught, or even if `try`/`catch` contains a `return` statement.

```javascript
function demo() {
  try {
    return "from try";
  } finally {
    console.log("finally still runs even with a return");
  }
}

console.log(demo());
// Output:
// finally still runs even with a return
// from try
```

---

### **The `throw` Statement**
`throw` immediately stops normal execution and hands control to the nearest enclosing `catch` block (or crashes the program if none exists).

#### **Throwing Custom Values vs Error Objects**
Technically, you can `throw` *any* value in JavaScript — a string, a number, an object — but throwing an actual `Error` object (or subclass) is strongly recommended because it automatically captures a stack trace and provides a consistent `.message`/`.name` interface.

```javascript
// Discouraged: throwing a plain value
function divideBad(a, b) {
  if (b === 0) {
    throw "Cannot divide by zero"; // No stack trace, inconsistent shape
  }
  return a / b;
}

// Recommended: throwing an Error object
function divideGood(a, b) {
  if (b === 0) {
    throw new Error("Cannot divide by zero"); // Has .message, .name, .stack
  }
  return a / b;
}

try {
  divideGood(10, 0);
} catch (error) {
  console.log(error.message); // Output: Cannot divide by zero
  console.log(error.name);    // Output: Error
  console.log(error instanceof Error); // Output: true
}
```

---

### **Built-in Error Types**
JavaScript provides several built-in error subclasses, each thrown automatically by the engine in specific situations.

1. **`Error`**: The generic base type all other error types inherit from. Used for custom or general-purpose errors.
2. **`TypeError`**: Thrown when a value is not of the expected type (e.g., calling a non-function, or accessing a property on `null`/`undefined`).
3. **`ReferenceError`**: Thrown when referencing a variable that doesn't exist or hasn't been declared.
4. **`SyntaxError`**: Thrown when code violates JavaScript's grammar rules — usually at parse time, but also by functions like `JSON.parse()` on malformed input.
5. **`RangeError`**: Thrown when a value is outside the range of what's allowed (e.g., an invalid array length or invalid number of decimal places).

```javascript
// TypeError
try {
  const num = 5;
  num(); // num is not a function
} catch (e) {
  console.log(e.name, "-", e.message);
  // Output: TypeError - num is not a function
}

// ReferenceError
try {
  console.log(undeclaredVariable);
} catch (e) {
  console.log(e.name, "-", e.message);
  // Output: ReferenceError - undeclaredVariable is not defined
}

// SyntaxError (via JSON.parse at runtime)
try {
  JSON.parse("{ bad json }");
} catch (e) {
  console.log(e.name, "-", e.message);
  // Output: SyntaxError - Unexpected token b in JSON at position 2
}

// RangeError
try {
  const arr = new Array(-1); // Invalid array length
} catch (e) {
  console.log(e.name, "-", e.message);
  // Output: RangeError - Invalid array length
}

// RangeError with toFixed
try {
  (10).toFixed(101); // Argument must be between 0 and 100
} catch (e) {
  console.log(e.name, "-", e.message);
  // Output: RangeError - toFixed() digits argument must be between 0 and 100
}
```

#### **Comparison Table**

| Error Type | Common Trigger |
|---|---|
| **`Error`** | Base class; used for generic or custom errors |
| **`TypeError`** | Calling something that isn't a function, using a value of the wrong type |
| **`ReferenceError`** | Accessing an undeclared variable |
| **`SyntaxError`** | Invalid code syntax, or malformed input to `JSON.parse()`/`eval()` |
| **`RangeError`** | A numeric value or length is outside its allowed range |

---

### **Custom Error Classes**
Extending the built-in `Error` class lets you create domain-specific error types with extra properties, while still behaving like a normal `Error` (with `.message`, `.stack`, `instanceof Error` support, etc.).

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);       // Set the standard .message property
    this.name = "ValidationError"; // Override the default "Error" name
    this.field = field;   // Add custom, domain-specific data
    Error.captureStackTrace?.(this, ValidationError); // Cleaner stack trace (V8 engines)
  }
}

function validateAge(age) {
  if (typeof age !== "number") {
    throw new ValidationError("Age must be a number", "age");
  }
  if (age < 0 || age > 120) {
    throw new ValidationError("Age must be between 0 and 120", "age");
  }
  return true;
}

try {
  validateAge(-5);
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`${error.name}: ${error.message} (field: ${error.field})`);
    // Output: ValidationError: Age must be between 0 and 120 (field: age)
  } else {
    throw error; // Re-throw anything we don't know how to handle
  }
}
```

This pattern lets calling code distinguish between different failure types using `instanceof`, enabling targeted handling (e.g., showing a form field error for `ValidationError` but logging and alerting for unexpected errors).

---

### **Error Handling in Async Code**

#### **`try`/`catch` with `async`/`await`**
```javascript
async function fetchUserData(id) {
  try {
    const response = await fetch(`https://api.example.com/users/${id}`);
    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch user:", error.message);
    throw error; // Optionally re-throw for the caller to handle further
  }
}
```

#### **`.catch()` with Promise Chains**
```javascript
fetch("https://api.example.com/users/1")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }
    return response.json();
  })
  .then((data) => console.log(data))
  .catch((error) => console.error("Failed to fetch user:", error.message));
```

#### **Comparison**

| Aspect | `try`/`catch` with `async`/`await` | `.catch()` with Promises |
|---|---|---|
| **Readability** | Linear, synchronous-looking | Chained, can nest awkwardly |
| **Catches sync AND async errors** | Yes, both in the same block | Only errors within the promise chain |
| **Error scoping** | One `try` can wrap multiple `await`s | Usually one `.catch()` per chain |
| **Style** | Imperative | Functional/declarative |

A crucial detail: inside an `async` function, both synchronous errors (like a `TypeError` thrown directly) and asynchronous errors (from a rejected `await`ed promise) are caught by the **same** `try`/`catch` block — this unification is one of `async`/`await`'s biggest advantages.

---

### **Global Error Handling (Browser)**
For errors that escape all local `try`/`catch` blocks, browsers provide global handlers as a last line of defense — typically used for logging/monitoring rather than recovery.

1. **`window.onerror`**: Fires for uncaught synchronous errors (runtime exceptions anywhere in the page's scripts).
   ```javascript
   window.onerror = function (message, source, lineno, colno, error) {
     console.log("Global error caught:", message);
     // Send to logging/monitoring service
     return true; // Prevents the default browser console error
   };
   ```

2. **`unhandledrejection`**: Fires when a Promise rejects and no `.catch()` or `try`/`catch` ever handles it.
   ```javascript
   window.addEventListener("unhandledrejection", function (event) {
     console.log("Unhandled promise rejection:", event.reason);
     event.preventDefault(); // Prevents default browser logging, if desired
   });

   Promise.reject(new Error("Oops, nobody caught this"));
   // Output: Unhandled promise rejection: Error: Oops, nobody caught this
   ```

These global handlers are best used for centralized error logging/monitoring (e.g., sending errors to a service like Sentry) — they should not be relied on as a substitute for proper `try`/`catch` handling closer to where errors actually occur.

---

### **Best Practices**
- Always throw `Error` objects (or subclasses), never plain strings or raw values, so consumers get a consistent `.message`, `.name`, and `.stack`.
- Create custom error classes for distinct failure categories (validation, network, authorization) so calling code can handle them differently via `instanceof`.
- Use `finally` for cleanup logic (closing connections, hiding loading spinners) that must run regardless of success or failure.
- In `async` functions, wrap `await` calls in `try`/`catch` rather than relying solely on `.catch()` chains, for more readable, unified error handling.
- Don't swallow errors silently — at minimum, log them; consider re-throwing when a function can't fully recover from an error itself.
- Use global handlers (`window.onerror`, `unhandledrejection`) as a safety net for logging/monitoring, not as your primary error-handling strategy.

---

### **Interview Questions**

**Q1. What is the execution order of `try`, `catch`, and `finally`?**
`try` runs first until it completes or throws. If it throws, `catch` runs next. `finally` always runs last, regardless of whether an error was thrown or caught, and even overrides a pending `return` value only if `finally` itself contains a `return` or `throw`.

**Q2. Why is it recommended to throw `Error` objects instead of plain strings or values?**
`Error` objects automatically capture useful metadata like `.message`, `.name`, and `.stack` (a stack trace), and support `instanceof Error` checks. Throwing a plain string loses all of that, making debugging and structured error handling much harder.

**Q3. What is the difference between a `TypeError` and a `ReferenceError`?**
A `TypeError` occurs when a value is used in a way that's incompatible with its type, such as calling something that isn't a function. A `ReferenceError` occurs when code tries to access a variable or identifier that doesn't exist in any accessible scope.

**Q4. When does a `RangeError` occur?**
It occurs when a value falls outside of an allowed range, such as creating an array with a negative length (`new Array(-1)`), or calling `toFixed()` with a digit count outside the 0–100 range.

**Q5. How do you create a custom error class in JavaScript?**
Extend the built-in `Error` class, call `super(message)` in the constructor to set the standard message, and optionally override `this.name` and add custom properties.
```javascript
class NotFoundError extends Error {
  constructor(resource) {
    super(`${resource} not found`);
    this.name = "NotFoundError";
  }
}
```

**Q6. How does error handling differ between `async`/`await` and `.then()`/`.catch()` chains?**
With `async`/`await`, both synchronous and asynchronous errors within the function are caught by a single `try`/`catch` block, giving linear, readable control flow. With `.then()`/`.catch()`, errors propagate down the promise chain and are caught by whichever `.catch()` comes next in the chain — synchronous errors outside the chain are not caught by it.

**Q7. What happens if you don't handle a rejected Promise at all?**
The rejection becomes an "unhandled promise rejection." In browsers, this triggers the global `unhandledrejection` event and typically logs a warning to the console; in Node.js, depending on version, it may log a warning or terminate the process.

**Q8. What is the purpose of the `finally` block, and does it run if there's a `return` inside `try`?**
`finally` is used for cleanup code that must run no matter the outcome — success, failure, or even an early `return`. Yes, it still executes even if `try` (or `catch`) contains a `return` statement; the `return` value is only used after `finally` finishes running.

**Q9. How would you distinguish between different types of custom errors when catching them?**
Use `instanceof` checks against your custom error classes (e.g., `if (error instanceof ValidationError)`), allowing you to branch handling logic based on the specific error subclass, rather than parsing error messages as strings.

**Q10. What does `window.onerror` capture, and what does it not capture?**
`window.onerror` captures uncaught synchronous runtime errors that bubble all the way up without being caught by any `try`/`catch`. It does not capture rejected Promises that are never handled — those require the separate `unhandledrejection` event instead.

**Q11. Is it good practice to catch every error at the point it occurs?**
Not always. Sometimes it's better to let an error propagate up to a caller that has enough context to decide how to recover (retry, show a UI message, log and exit), rather than catching and silently ignoring it too early, which can hide real bugs.

**Q12. What is the difference between `SyntaxError` thrown at parse time versus at runtime (e.g., via `JSON.parse`)?**
A parse-time `SyntaxError` happens when the JavaScript engine cannot even parse your source code into valid syntax — this halts execution before any code runs and cannot be caught by `try`/`catch`. A runtime `SyntaxError`, such as one thrown by `JSON.parse()` on malformed JSON text, occurs while the program is already running and *can* be caught normally with `try`/`catch`.
