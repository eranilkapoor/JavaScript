**Interpolation in JavaScript** refers to embedding variables, expressions, or values into strings using a template literal. It is a powerful feature introduced in **ES6** that makes string concatenation more readable and expressive compared to older methods.

---

### **Syntax**
Interpolation uses **template literals**, denoted by backticks (`` ` ``). Variables or expressions are embedded within the string using the syntax `${expression}`.

---

### **Example: Interpolating Variables**
```javascript
const name = "John";
const age = 30;

const greeting = `Hello, my name is ${name} and I am ${age} years old.`;

console.log(greeting);
// Output: Hello, my name is John and I am 30 years old.
```

---

### **Example: Interpolating Expressions**
You can embed any JavaScript expression inside `${}`:
```javascript
const a = 5;
const b = 10;

const result = `The sum of ${a} and ${b} is ${a + b}.`;

console.log(result);
// Output: The sum of 5 and 10 is 15.
```

---

### **Features of Interpolation**
1. **Multiline Strings**: Template literals allow multi-line strings without needing escape characters like `\n`.
   ```javascript
   const message = `This is a
   multi-line
   string.`;

   console.log(message);
   // Output:
   // This is a
   // multi-line
   // string.
   ```

2. **Function Calls**: You can call functions directly inside `${}`.
   ```javascript
   const getName = () => "Alice";

   const message = `Hello, ${getName()}!`;

   console.log(message);
   // Output: Hello, Alice!
   ```

3. **Nested Templates**: Template literals can be nested.
   ```javascript
   const x = 2;
   const y = 3;

   const nested = `Sum: ${x + y}, Double: ${`${x * 2}, ${y * 2}`}`;
   console.log(nested);
   // Output: Sum: 5, Double: 4, 6
   ```

---

### **Comparison with String Concatenation**
**Old way (pre-ES6)**:
```javascript
const name = "John";
const age = 30;
const greeting = "Hello, my name is " + name + " and I am " + age + " years old.";

console.log(greeting);
// Output: Hello, my name is John and I am 30 years old.
```

**New way (with interpolation)**:
```javascript
const name = "John";
const age = 30;
const greeting = `Hello, my name is ${name} and I am ${age} years old.`;

console.log(greeting);
// Output: Hello, my name is John and I am 30 years old.
```
The new approach is cleaner and easier to read.

---

### **Use Cases**
1. **Dynamic Strings**: Useful for generating user messages or HTML content dynamically.
2. **Formatting Output**: Great for injecting dynamic values into logs or UI.
3. **Dynamic Function Calls**: Helps when constructing SQL queries or other strings with function output.

---

### **Limitations**
1. **Potential Security Risks**: If used with untrusted data, it can lead to **injection vulnerabilities** (e.g., in SQL queries).
2. **Limited to Strings**: Useful only for string operations and not a replacement for other data manipulation methods.

Interpolation is an essential feature for modern JavaScript development and enhances the readability and flexibility of string manipulation.