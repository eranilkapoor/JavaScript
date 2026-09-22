**Operators** let you perform computations, comparisons, and logical decisions on values, while **control structures** determine the order in which your code executes. Together they form the backbone of program logic in JavaScript.

---

### **Arithmetic Operators**
```javascript
console.log(10 + 3); // Output: 13
console.log(10 - 3); // Output: 7
console.log(10 * 3); // Output: 30
console.log(10 / 3); // Output: 3.3333333333333335
console.log(10 % 3); // Output: 1  (remainder)
console.log(10 ** 3); // Output: 1000  (exponentiation, ES2016)

let count = 5;
console.log(count++); // Output: 5 (returns then increments — "post-increment")
console.log(count);   // Output: 6
console.log(++count); // Output: 7 (increments then returns — "pre-increment")
```

---

### **Assignment Operators**
```javascript
let x = 10;
x += 5;  // x = x + 5   → 15
x -= 3;  // x = x - 3   → 12
x *= 2;  // x = x * 2   → 24
x /= 4;  // x = x / 4   → 6
x %= 4;  // x = x % 4   → 2
x **= 3; // x = x ** 3  → 8

console.log(x); // Output: 8
```

---

### **Comparison Operators: `==` vs `===`**
`==` (loose equality) coerces types before comparing; `===` (strict equality) compares value and type together, with no coercion.
```javascript
console.log(5 == "5");   // Output: true  (coerced)
console.log(5 === "5");  // Output: false (different types)

console.log(5 != "5");   // Output: false
console.log(5 !== "5");  // Output: true

console.log(10 > 5);   // Output: true
console.log(10 <= 10); // Output: true
```
As a rule of thumb, always prefer `===`/`!==` to avoid coercion surprises (see `VariablesAndDataTypes.md` for a deeper list of `==` gotchas).

---

### **Logical Operators**
1. **`&&` (AND)**: Returns the first falsy value, or the last value if all are truthy.
2. **`||` (OR)**: Returns the first truthy value, or the last value if all are falsy.
3. **`!` (NOT)**: Inverts a boolean value.

```javascript
console.log(true && "Hello");  // Output: Hello
console.log(0 && "Hello");     // Output: 0 (short-circuits, stops at first falsy)

console.log(false || "Default"); // Output: Default
console.log("Value" || "Default"); // Output: Value

console.log(!true);  // Output: false
console.log(!0);     // Output: true
```

#### **Short-Circuit Evaluation**
Because `&&` and `||` stop evaluating as soon as the result is determined, they're commonly used for conditional execution and default values:
```javascript
const user = { loggedIn: true, name: "Alice" };

user.loggedIn && console.log(`Welcome, ${user.name}`);
// Output: Welcome, Alice  (right side only runs because left side is truthy)

function greet(name) {
  name = name || "Guest"; // fallback if name is falsy
  console.log(`Hello, ${name}`);
}
greet(); // Output: Hello, Guest
```

---

### **Ternary Operator**
A compact one-line alternative to `if/else` for simple conditional expressions.
```javascript
const age = 20;
const status = age >= 18 ? "Adult" : "Minor";
console.log(status); // Output: Adult
```

---

### **Nullish Coalescing (`??`)**
Returns the right-hand value **only** when the left-hand value is `null` or `undefined` — unlike `||`, it does not treat other falsy values (`0`, `""`, `false`, `NaN`) as missing.
```javascript
const count1 = 0;
console.log(count1 || 10); // Output: 10  (0 is falsy, so || falls through — often wrong!)
console.log(count1 ?? 10); // Output: 0   (0 is not null/undefined, so ?? keeps it)

let username;
console.log(username ?? "Anonymous"); // Output: Anonymous
```

---

### **Optional Chaining (`?.`)**
Safely accesses deeply nested properties or calls methods without throwing an error if an intermediate value is `null` or `undefined`.
```javascript
const user = {
  profile: {
    address: { city: "New York" },
  },
};

console.log(user.profile?.address?.city);   // Output: New York
console.log(user.profile?.contact?.phone);  // Output: undefined (no error thrown)

// Works with function calls too
console.log(user.getGreeting?.()); // Output: undefined (skipped, method doesn't exist)

// Works with array access
const arr = null;
console.log(arr?.[0]); // Output: undefined
```

`??` and `?.` are frequently combined:
```javascript
const city = user.profile?.address?.city ?? "Unknown City";
console.log(city); // Output: New York
```

---

### **Type Casting / Explicit Conversion**
```javascript
// To Number
console.log(Number("42"));     // Output: 42
console.log(Number("42px"));   // Output: NaN
console.log(Number(true));     // Output: 1
console.log(parseInt("42px")); // Output: 42  (parses until it hits a non-digit)
console.log(parseFloat("3.14abc")); // Output: 3.14

// To String
console.log(String(42));    // Output: "42"
console.log(String(true));  // Output: "true"
console.log((42).toString()); // Output: "42"

// To Boolean
console.log(Boolean(0));     // Output: false
console.log(Boolean(""));    // Output: false
console.log(Boolean(null));  // Output: false
console.log(Boolean("0"));   // Output: true  (non-empty string is truthy!)
console.log(Boolean([]));    // Output: true  (empty array/object is truthy)
```

| Value | `Boolean(value)` |
|---|---|
| `0`, `-0`, `NaN` | `false` |
| `""` (empty string) | `false` |
| `null`, `undefined` | `false` |
| Everything else (including `"0"`, `[]`, `{}`) | `true` |

---

### **Control Structures**

#### **`if / else`**
```javascript
const temperature = 15;

if (temperature > 30) {
  console.log("Hot");
} else if (temperature > 15) {
  console.log("Warm");
} else {
  console.log("Cold");
}
// Output: Cold
```

#### **`switch` (and the Fallthrough Gotcha)**
`switch` compares using strict equality (`===`). Forgetting `break` causes execution to **fall through** into the next case.
```javascript
const day = 2;

switch (day) {
  case 1:
    console.log("Monday");
    break;
  case 2:
    console.log("Tuesday");
  // no break here! falls through
  case 3:
    console.log("Wednesday");
    break;
  default:
    console.log("Unknown day");
}
// Output:
// Tuesday
// Wednesday   <-- fallthrough executed unintentionally
```
Always include `break` (or `return` inside a function) unless intentionally grouping cases:
```javascript
const fruit = "apple";
switch (fruit) {
  case "apple":
  case "pear":
    console.log("Common fruit"); // intentional fallthrough — grouping cases
    break;
  default:
    console.log("Exotic fruit");
}
// Output: Common fruit
```

#### **`for` Loop**
```javascript
for (let i = 0; i < 3; i++) {
  console.log(i);
}
// Output: 0
//         1
//         2
```

#### **`while` Loop**
```javascript
let n = 0;
while (n < 3) {
  console.log(n);
  n++;
}
// Output: 0
//         1
//         2
```

#### **`do...while` Loop**
Executes the body **at least once**, since the condition is checked after the first run.
```javascript
let m = 5;
do {
  console.log(m);
  m++;
} while (m < 3);
// Output: 5  (runs once even though 5 < 3 is false)
```

#### **`for...of` vs `for...in`**
- **`for...of`**: Iterates over the **values** of an iterable (arrays, strings, Maps, Sets). Best for arrays.
- **`for...in`**: Iterates over the **enumerable property keys** of an object (including inherited ones). Best for plain objects, but risky on arrays.

```javascript
const arr = ["a", "b", "c"];

for (const value of arr) {
  console.log(value); // Output: a, b, c
}

for (const index in arr) {
  console.log(index); // Output: "0", "1", "2" (indices as strings, not values!)
}

const obj = { x: 1, y: 2 };
for (const key in obj) {
  console.log(key, obj[key]); // Output: x 1 | y 2
}
// for...of obj would throw: TypeError: obj is not iterable
```

| Aspect | `for...of` | `for...in` |
|---|---|---|
| Iterates over | Values of an iterable | Enumerable keys (property names) |
| Works on | Arrays, Strings, Maps, Sets | Objects (also arrays, but not recommended) |
| Array index type | N/A (gives values directly) | String indices `"0"`, `"1"` |
| Inherited properties | Not applicable | Includes inherited enumerable properties |

#### **`break` and `continue`**
```javascript
for (let i = 0; i < 5; i++) {
  if (i === 3) break; // exits the loop entirely
  console.log(i);
}
// Output: 0, 1, 2

for (let i = 0; i < 5; i++) {
  if (i === 2) continue; // skips this iteration only
  console.log(i);
}
// Output: 0, 1, 3, 4
```

#### **Labeled Statements**
Labels let `break`/`continue` target an outer loop from within a nested loop.
```javascript
outerLoop: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outerLoop; // skips to next outer iteration
    console.log(`i=${i}, j=${j}`);
  }
}
// Output:
// i=0, j=0
// i=1, j=0
// i=2, j=0
```

---

### **Best Practices**
- Always use `===`/`!==` unless coercion is intentional and understood.
- Prefer `??` over `||` when you want to preserve legitimate falsy values like `0` or `""`.
- Use optional chaining (`?.`) to avoid verbose manual null-checks on nested data.
- Always add `break` in `switch` statements unless you explicitly intend to fall through — and add a comment when you do.
- Prefer `for...of` for arrays and iterables; reserve `for...in` for plain objects.
- Avoid deeply nested labeled loops where possible — refactor into functions for readability.

---

### **Interview Questions**

**Q1. What's the difference between `==` and `===`?**
`==` performs type coercion before comparing values, which can produce unintuitive results (`"5" == 5` is `true`). `===` compares both type and value with no coercion (`"5" === 5` is `false`), which is why it's the recommended default.

**Q2. How does short-circuit evaluation work with `&&` and `||`?**
`&&` stops and returns the first falsy operand (or the last value if all are truthy); `||` stops and returns the first truthy operand (or the last value if all are falsy). This lets them be used for conditional execution and default value assignment.

**Q3. What's the difference between `??` and `||`?**
`||` returns the right-hand value if the left is any falsy value (`0`, `""`, `false`, `null`, `undefined`, `NaN`). `??` only falls back when the left-hand value is specifically `null` or `undefined`, preserving legitimate falsy values like `0`.

**Q4. What does optional chaining (`?.`) do?**
It safely accesses a property, array index, or method on a potentially `null`/`undefined` value, short-circuiting to `undefined` instead of throwing a `TypeError`, e.g., `user.profile?.address?.city`.

**Q5. What is the switch fallthrough gotcha?**
If a `case` block doesn't end with `break` (or `return`), execution "falls through" and continues executing the code in the next `case` block regardless of whether its condition matches, which is a common source of bugs.

**Q6. What's the difference between `for...of` and `for...in`?**
`for...of` iterates over the values of an iterable (arrays, strings, Maps, Sets). `for...in` iterates over the enumerable property keys of an object, including inherited ones, and on arrays it yields string indices rather than values — which is why it's discouraged for arrays.

**Q7. What's the difference between `parseInt("42px")` and `Number("42px")`?**
`parseInt` parses the string from the left until it hits a non-numeric character, so it returns `42`. `Number` requires the entire string to be a valid numeric representation, so it returns `NaN` because of the trailing `"px"`.

**Q8. What's the difference between `break` and `continue`?**
`break` exits the enclosing loop (or switch) entirely. `continue` skips only the current iteration and proceeds to the next one.

**Q9. What values are falsy in JavaScript?**
`false`, `0`, `-0`, `""` (empty string), `null`, `undefined`, and `NaN`. Everything else, including `"0"`, `[]`, and `{}`, is truthy.

**Q10. When would you use a labeled statement?**
When you need `break` or `continue` to affect an outer loop from inside a nested loop — e.g., `continue outerLoop` skips to the next iteration of the outer loop instead of the inner one. It's used sparingly since it can hurt readability.

**Q11. Why does `do...while` run at least once even if the condition is false?**
Because the condition check happens **after** the loop body executes for the first time, unlike a regular `while` loop, which checks the condition before running the body at all.

**Q12. Why is `0 || "default"` often a bug, and how do you fix it?**
If `0` is a legitimate value you want to keep (e.g., a score or count), `||` incorrectly treats it as missing and returns `"default"`. Using `??` instead (`0 ?? "default"`) correctly returns `0` since `??` only triggers on `null`/`undefined`.
