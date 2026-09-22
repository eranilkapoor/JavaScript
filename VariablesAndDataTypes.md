**Variables** are named containers used to store data that a program can reference and manipulate later. JavaScript gives you three ways to declare a variable — `var`, `let`, and `const` — and understanding their differences in scope, hoisting, and re-assignment is fundamental to writing predictable code.

---

### **`var` vs `let` vs `const`**

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisting | Hoisted & initialized as `undefined` | Hoisted but **not** initialized (TDZ) | Hoisted but **not** initialized (TDZ) |
| Re-declaration | Allowed | Not allowed (same scope) | Not allowed (same scope) |
| Re-assignment | Allowed | Allowed | **Not allowed** |
| Attaches to `window`/`global` | Yes (in browsers, global scope) | No | No |

#### **1. Scope**
`var` is **function-scoped** — it ignores block boundaries like `if` or `for` and is visible anywhere inside the enclosing function. `let` and `const` are **block-scoped** — confined to the nearest `{ }`.
```javascript
function scopeDemo() {
  if (true) {
    var functionScoped = "I leak out of the block";
    let blockScoped = "I stay inside the block";
  }
  console.log(functionScoped); // Output: I leak out of the block
  console.log(blockScoped);    // ReferenceError: blockScoped is not defined
}
```

#### **2. Hoisting**
All variable declarations are **hoisted** (moved to the top of their scope during compilation), but they behave differently:
- `var` declarations are hoisted **and initialized with `undefined`**, so accessing them before the declaration line gives `undefined` instead of an error.
- `let` and `const` are hoisted but **not initialized** — they remain in the **Temporal Dead Zone (TDZ)** from the start of the scope until the declaration line is executed.

```javascript
console.log(a); // Output: undefined (hoisted, not yet assigned)
var a = 10;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 20;
```

#### **3. Temporal Dead Zone (TDZ)**
The TDZ is the period between entering a scope and the point where a `let`/`const` variable is actually declared. Accessing the variable during this window throws a `ReferenceError`, which helps catch bugs caused by using a variable before it's meant to be used.
```javascript
{
  // TDZ for `x` starts here
  console.log(typeof x); // ReferenceError (not "undefined"!)
  let x = 5;
  // TDZ ends here
}
```

#### **4. Re-declaration and Re-assignment**
```javascript
var v = 1;
var v = 2; // fine, re-declaration allowed
console.log(v); // Output: 2

let l = 1;
// let l = 2; // SyntaxError: Identifier 'l' has already been declared
l = 2; // fine, re-assignment allowed
console.log(l); // Output: 2

const c = 1;
// c = 2; // TypeError: Assignment to constant variable.
```

> **Note**: `const` prevents re-assignment of the variable binding, not mutation of the value. An object or array declared with `const` can still have its properties/elements changed.
```javascript
const person = { name: "Alice" };
person.name = "Bob"; // allowed — mutating the object, not reassigning `person`
console.log(person); // Output: { name: 'Bob' }
```

#### **5. Classic Hoisting Pitfall: `var` in Loops**
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 3, 3, 3  (all callbacks share the same function-scoped `i`)

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// Output: 0, 1, 2  (each iteration gets its own block-scoped `j`)
```

---

### **Primitive Data Types**
JavaScript has **7 primitive types**. Primitives are immutable and compared **by value**.

1. **Number**: Represents both integers and floating-point numbers (JavaScript has no separate `int`/`float`).
   ```javascript
   const age = 30;
   const price = 19.99;
   console.log(typeof age); // Output: number
   ```
2. **String**: Textual data, written with single quotes, double quotes, or backticks.
   ```javascript
   const name = "Alice";
   console.log(typeof name); // Output: string
   ```
3. **Boolean**: `true` or `false`.
   ```javascript
   const isActive = true;
   console.log(typeof isActive); // Output: boolean
   ```
4. **Undefined**: A variable that has been declared but has not been assigned a value.
   ```javascript
   let notAssigned;
   console.log(notAssigned);      // Output: undefined
   console.log(typeof notAssigned); // Output: undefined
   ```
5. **Null**: Represents the intentional absence of any value.
   ```javascript
   let empty = null;
   console.log(typeof empty); // Output: object  (a famous long-standing JS quirk)
   ```
6. **Symbol** (ES6): Creates a unique, immutable identifier, often used as a hidden/unique object property key.
   ```javascript
   const id1 = Symbol("id");
   const id2 = Symbol("id");
   console.log(id1 === id2); // Output: false (every symbol is unique)
   console.log(typeof id1);  // Output: symbol
   ```
7. **BigInt** (ES2020): Represents integers larger than `Number.MAX_SAFE_INTEGER` (2^53 - 1), created by appending `n` to a numeric literal.
   ```javascript
   const bigNumber = 9007199254740993n;
   console.log(typeof bigNumber); // Output: bigint
   ```

---

### **Reference (Non-Primitive) Types**
Everything that isn't a primitive is an **object** under the hood. Reference types are stored and compared **by reference** (by memory address), not by value.

1. **Object**: A collection of key-value pairs.
   ```javascript
   const car = { brand: "Toyota", year: 2024 };
   console.log(typeof car); // Output: object
   ```
2. **Array**: An ordered, index-based list — technically a specialized object.
   ```javascript
   const fruits = ["apple", "banana", "cherry"];
   console.log(typeof fruits);        // Output: object
   console.log(Array.isArray(fruits)); // Output: true
   ```
3. **Function**: Functions are also objects (callable objects) in JavaScript.
   ```javascript
   function greet() {}
   console.log(typeof greet); // Output: function
   ```

```javascript
// Reference types compare by reference, not value
const obj1 = { value: 1 };
const obj2 = { value: 1 };
console.log(obj1 === obj2); // Output: false (different objects in memory)

const obj3 = obj1;
console.log(obj1 === obj3); // Output: true (same reference)
```

---

### **`undefined` vs `null`**

| Aspect | `undefined` | `null` |
|---|---|---|
| Meaning | Variable declared but not yet assigned | Intentional "no value" set by the developer |
| Set by | JavaScript automatically | Explicitly by the programmer |
| Type (`typeof`) | `"undefined"` | `"object"` (a historical bug kept for compatibility) |
| Equality (`==`) | `undefined == null` → `true` | `null == undefined` → `true` |
| Equality (`===`) | `undefined === null` → `false` | `undefined === null` → `false` |

```javascript
let a;
let b = null;

console.log(a); // Output: undefined
console.log(b); // Output: null
console.log(a == b);  // Output: true  (loose equality treats them as equal)
console.log(a === b); // Output: false (different types)
```

---

### **Type Coercion vs Type Conversion**

**Type conversion** is explicit — you intentionally convert a value from one type to another. **Type coercion** is implicit — JavaScript automatically converts types behind the scenes, often during operations like `+`, `==`, or `if` conditions.

```javascript
// Explicit conversion
const str = String(123);   // "123"
const num = Number("456"); // 456

// Implicit coercion
console.log("5" + 3);   // Output: "53"  (number coerced to string, concatenation)
console.log("5" - 3);   // Output: 2     (string coerced to number, subtraction)
console.log("5" * "2"); // Output: 10    (both coerced to numbers)
console.log(1 + true);  // Output: 2     (true coerced to 1)
console.log("" + []);   // Output: ""    (empty array coerced to empty string)
```

#### **`==` vs `===` Surprises**
`==` (loose equality) performs type coercion before comparing; `===` (strict equality) compares both value **and** type without conversion.
```javascript
console.log(0 == false);      // Output: true  (coercion)
console.log(0 === false);     // Output: false (different types)

console.log("" == 0);         // Output: true  (coercion)
console.log("" === 0);        // Output: false

console.log(null == undefined);  // Output: true
console.log(null === undefined); // Output: false

console.log(NaN == NaN);      // Output: false (NaN is never equal to anything, even itself)
console.log([] == false);     // Output: true  ([] → "" → 0, false → 0)
```
Because of these edge cases, it's a strong convention to **always use `===` / `!==`** unless you have a specific, well-understood reason to use `==`.

---

### **Strict Mode**
`"use strict"` opts your code into a restricted variant of JavaScript that catches common mistakes and unsafe patterns, either for an entire script or a single function, by placing the string at the top.

```javascript
"use strict";

x = 10; // ReferenceError: x is not defined (undeclared variables are disallowed)
```

**What strict mode changes:**
1. **No accidental globals**: Assigning to an undeclared variable throws a `ReferenceError` instead of silently creating a global variable.
2. **Silent failures become errors**: Assigning to a read-only property, or a non-writable/non-extensible object, throws instead of failing silently.
   ```javascript
   "use strict";
   const obj = Object.freeze({ x: 1 });
   obj.x = 2; // TypeError in strict mode (silently ignored otherwise)
   ```
3. **`this` is `undefined`** in a regular function called without a receiver, instead of defaulting to the global object.
   ```javascript
   "use strict";
   function showThis() {
     console.log(this);
   }
   showThis(); // Output: undefined
   ```
4. **Disallows duplicate parameter names** and some legacy syntax like `with`.
5. **ES modules and classes are strict by default** — you don't need to add the directive manually inside them.

---

### **Best Practices**
- Default to `const`; use `let` only when a variable genuinely needs to be reassigned; avoid `var` in modern code.
- Always initialize variables when you declare them to avoid confusing `undefined` states.
- Use `===`/`!==` instead of `==`/`!=` unless coercion is explicitly desired.
- Use `null` to intentionally signal "no value," and let `undefined` represent "not yet assigned" — don't mix the two arbitrarily.
- Enable strict mode (or use ES modules, which are strict automatically) in all your scripts.
- Prefer explicit type conversion (`Number()`, `String()`, `Boolean()`) over relying on implicit coercion for clarity.

---

### **Interview Questions**

**Q1. What are the key differences between `var`, `let`, and `const`?**
`var` is function-scoped, hoisted and initialized as `undefined`, and can be re-declared and reassigned. `let` and `const` are block-scoped and live in the Temporal Dead Zone until declared; `let` can be reassigned but not re-declared in the same scope, while `const` can be neither reassigned nor re-declared.

**Q2. What is the Temporal Dead Zone (TDZ)?**
It's the span between entering a scope and the line where a `let`/`const` variable is actually declared. Accessing the variable in that window throws a `ReferenceError`, unlike `var`, which just returns `undefined`.

**Q3. Why does `const` allow mutating an object's properties?**
Because `const` only makes the **variable binding** immutable — it prevents reassigning the variable to a new value — but it doesn't freeze the object itself. To prevent mutation of the object's contents, use `Object.freeze()`.

**Q4. What is the difference between `undefined` and `null`?**
`undefined` means a variable has been declared but not assigned a value, and JavaScript sets it automatically. `null` is an intentional, explicit assignment made by the developer to represent "no value." They are loosely equal (`==`) but not strictly equal (`===`), because they're different types.

**Q5. Why does `typeof null` return `"object"`?**
It's a legacy bug from the original JavaScript implementation in 1995, where values were represented with a type tag, and `null`'s tag happened to match the object tag. It's kept for backward compatibility even though it's technically incorrect.

**Q6. What's the difference between type coercion and type conversion?**
Type conversion is explicit — the developer deliberately converts a value's type using functions like `Number()` or `String()`. Type coercion is implicit — JavaScript automatically converts types behind the scenes during operations like `+`, `==`, or in boolean contexts.

**Q7. Why should you prefer `===` over `==`?**
`==` performs type coercion before comparing, which can lead to unintuitive results like `"" == 0` being `true` or `[] == false` being `true`. `===` compares both value and type without coercion, making comparisons predictable.

**Q8. What does `"use strict"` actually do?**
It enables a stricter parsing and execution mode that throws errors for unsafe actions — such as assigning to undeclared variables, duplicate parameters, or writing to frozen objects — that would otherwise fail silently or create implicit globals.

**Q9. What are the primitive types in JavaScript?**
Number, String, Boolean, Undefined, Null, Symbol, and BigInt. Everything else (objects, arrays, functions) is a reference type.

**Q10. How are primitives different from reference types when passed around or compared?**
Primitives are compared and copied by value — each variable holds an independent copy. Reference types (objects, arrays, functions) are compared and copied by reference — variables hold a pointer to the same underlying data in memory.

**Q11. What happens if you try to re-declare a `let` variable in the same scope?**
JavaScript throws a `SyntaxError: Identifier 'x' has already been declared`, unlike `var`, which silently allows re-declaration.

**Q12. What is variable hoisting?**
Hoisting is JavaScript's behavior of moving variable and function declarations to the top of their containing scope during the compilation phase, before code execution. `var` declarations are hoisted and initialized as `undefined`; `let`/`const` are hoisted but stay uninitialized in the TDZ; function declarations are hoisted along with their full definition.
