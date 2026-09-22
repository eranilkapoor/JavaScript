**Arrays** are ordered, index-based collections used to store multiple values in a single variable. JavaScript arrays are dynamic (they can grow/shrink), can hold mixed types, and come with a rich set of built-in methods for adding, removing, searching, and transforming data.

---

### **Creating Arrays**

```javascript
// 1. Array literal (most common)
const fruits = ["apple", "banana", "cherry"];

// 2. Array constructor
const numbers = new Array(1, 2, 3);
console.log(numbers); // Output: [ 1, 2, 3 ]

// Careful: a single numeric argument creates an empty array of that length!
const empty = new Array(5);
console.log(empty); // Output: [ <5 empty items> ]
console.log(empty.length); // Output: 5

// 3. Array.of() — always creates an array from its arguments, avoiding the constructor quirk
const single = Array.of(5);
console.log(single); // Output: [ 5 ]

// 4. Array.from() — builds an array from an iterable or array-like object
const fromString = Array.from("hello");
console.log(fromString); // Output: [ 'h', 'e', 'l', 'l', 'o' ]

const fromRange = Array.from({ length: 5 }, (_, i) => i * 2);
console.log(fromRange); // Output: [ 0, 2, 4, 6, 8 ]
```

---

### **Mutating Methods**
These methods **change the original array**.

```javascript
const arr = [1, 2, 3];

arr.push(4);        // adds to the end
console.log(arr);   // Output: [ 1, 2, 3, 4 ]

arr.pop();           // removes from the end
console.log(arr);    // Output: [ 1, 2, 3 ]

arr.unshift(0);       // adds to the beginning
console.log(arr);     // Output: [ 0, 1, 2, 3 ]

arr.shift();           // removes from the beginning
console.log(arr);      // Output: [ 1, 2, 3 ]

// splice(start, deleteCount, ...itemsToInsert)
const letters = ["a", "b", "c", "d"];
letters.splice(1, 2, "x", "y", "z"); // remove 2 items at index 1, insert 3
console.log(letters); // Output: [ 'a', 'x', 'y', 'z', 'd' ]

const nums = [3, 1, 4, 1, 5];
nums.sort(); // default sort is lexicographic (string-based)!
console.log(nums); // Output: [ 1, 1, 3, 4, 5 ]

const nums2 = [10, 2, 33, 4];
nums2.sort(); // WRONG for numbers without a comparator
console.log(nums2); // Output: [ 10, 2, 33, 4 ] -> sorted as strings

nums2.sort((a, b) => a - b); // correct numeric sort
console.log(nums2); // Output: [ 2, 4, 10, 33 ]

const seq = [1, 2, 3];
seq.reverse();
console.log(seq); // Output: [ 3, 2, 1 ]
```

---

### **Non-Mutating Methods**
These methods return a **new array or value**, leaving the original untouched.

```javascript
const arr = [1, 2, 3, 4, 5];

const sliced = arr.slice(1, 3); // start (inclusive) to end (exclusive)
console.log(sliced); // Output: [ 2, 3 ]
console.log(arr);    // Output: [ 1, 2, 3, 4, 5 ] (unchanged)

const combined = [1, 2].concat([3, 4], [5]);
console.log(combined); // Output: [ 1, 2, 3, 4, 5 ]

const joined = ["2024", "01", "15"].join("-");
console.log(joined); // Output: 2024-01-15
```

---

### **Iteration and Transformation Methods**

```javascript
const numbers = [1, 2, 3, 4, 5];

// forEach — runs a function for each element, returns undefined
numbers.forEach((n) => console.log(n * 2));
// Output: 2, 4, 6, 8, 10 (logged individually)

// map — transforms each element, returns a NEW array
const doubled = numbers.map((n) => n * 2);
console.log(doubled); // Output: [ 2, 4, 6, 8, 10 ]

// filter — keeps elements that pass a test, returns a NEW array
const evens = numbers.filter((n) => n % 2 === 0);
console.log(evens); // Output: [ 2, 4 ]

// reduce — accumulates a single value from all elements
const total = numbers.reduce((acc, n) => acc + n, 0);
console.log(total); // Output: 15

// find — returns the FIRST matching element (or undefined)
const firstEven = numbers.find((n) => n % 2 === 0);
console.log(firstEven); // Output: 2

// findIndex — returns the index of the first match (or -1)
const firstEvenIndex = numbers.findIndex((n) => n % 2 === 0);
console.log(firstEvenIndex); // Output: 1

// some — true if AT LEAST ONE element passes the test
console.log(numbers.some((n) => n > 4)); // Output: true

// every — true only if ALL elements pass the test
console.log(numbers.every((n) => n > 0)); // Output: true

// includes — true if the array contains the given value
console.log(numbers.includes(3)); // Output: true
console.log(numbers.includes(10)); // Output: false
```

#### **`map` vs `forEach`**
```javascript
const a = [1, 2, 3];
const result1 = a.forEach((n) => n * 2);
console.log(result1); // Output: undefined (forEach doesn't return anything useful)

const result2 = a.map((n) => n * 2);
console.log(result2); // Output: [ 2, 4, 6 ]
```
Use `map` when you need a transformed array; use `forEach` when you're just performing side effects (like logging or pushing to an external array).

---

### **Destructuring Arrays**
Extracts values from an array into individual variables based on position.
```javascript
const coords = [10, 20, 30];
const [x, y, z] = coords;
console.log(x, y, z); // Output: 10 20 30

// Skipping elements
const [first, , third] = coords;
console.log(first, third); // Output: 10 30

// Default values
const [a = 1, b = 2, c = 3, d = 4] = [10, 20];
console.log(a, b, c, d); // Output: 10 20 3 4

// Swapping variables without a temp variable
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q); // Output: 2 1
```

---

### **Spread Operator with Arrays**
Expands an iterable into individual elements — useful for copying, merging, and passing elements as arguments.
```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const merged = [...arr1, ...arr2];
console.log(merged); // Output: [ 1, 2, 3, 4, 5, 6 ]

const copy = [...arr1]; // shallow copy
console.log(copy); // Output: [ 1, 2, 3 ]

console.log(Math.max(...arr1)); // Output: 3 (spreads array into arguments)
```

---

### **Rest with Arrays**
Collects the remaining elements into a new array (the inverse of spread), commonly used in destructuring.
```javascript
const [head, ...tail] = [1, 2, 3, 4];
console.log(head); // Output: 1
console.log(tail);  // Output: [ 2, 3, 4 ]

function sumAll(...nums) {
  return nums.reduce((total, n) => total + n, 0);
}
console.log(sumAll(1, 2, 3, 4)); // Output: 10
```

---

### **Copying Arrays: Shallow Copy Pitfalls**
Because arrays are reference types, simply assigning one array to another variable does **not** create an independent copy.
```javascript
const original = [1, 2, 3];
const notACopy = original; // just another reference to the SAME array
notACopy.push(4);
console.log(original); // Output: [ 1, 2, 3, 4 ] (original was mutated too!)
```

**Proper (shallow) copy techniques:**
```javascript
const original2 = [1, 2, 3];

const copy1 = [...original2];         // spread operator
const copy2 = original2.slice();      // slice with no arguments
const copy3 = Array.from(original2);  // Array.from

copy1.push(4);
console.log(original2); // Output: [ 1, 2, 3 ] (unaffected)
```

**Shallow copy pitfall with nested objects/arrays:**
```javascript
const nested = [{ id: 1 }, { id: 2 }];
const shallowCopy = [...nested];

shallowCopy[0].id = 999; // mutates the SAME inner object
console.log(nested[0].id); // Output: 999 (also changed! only the top level was copied)
```
For a true independent copy of nested structures, use `structuredClone(original)` (modern environments) or a deep-clone utility/library.

---

### **Multidimensional Arrays**
JavaScript doesn't have true multidimensional arrays — instead, you nest arrays inside arrays.
```javascript
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

console.log(matrix[1][2]); // Output: 6 (row 1, column 2)

// Flattening a nested array
console.log(matrix.flat()); // Output: [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]

// Iterating a 2D array
matrix.forEach((row) => {
  row.forEach((value) => console.log(value));
});
```

---

### **Best Practices**
- Prefer non-mutating methods (`map`, `filter`, `slice`, spread) over mutating ones when working with state (especially in frameworks like React) to avoid unexpected side effects.
- Always provide a comparator function to `.sort()` when sorting numbers — the default sort is lexicographic.
- Use `Array.isArray()` instead of `typeof` to check if a value is an array, since `typeof [] === "object"`.
- Use `const` for array declarations by default — you can still mutate contents, but the variable can't be reassigned.
- Use `structuredClone()` or a dedicated deep-clone utility when copying arrays containing nested objects/arrays.
- Reach for `reduce` when you need to build up a single aggregate value; don't force it for simple transformations better suited to `map`/`filter`.

---

### **Interview Questions**

**Q1. What's the difference between `map` and `forEach`?**
`map` returns a new array containing the transformed values and doesn't mutate the original. `forEach` returns `undefined` and is used purely for side effects (like logging), not for producing a new array.

**Q2. Why does `[10, 2, 33, 4].sort()` produce an incorrect numeric order?**
Because `Array.prototype.sort()` converts elements to strings and sorts lexicographically by default. To sort numbers correctly, you must pass a comparator function: `.sort((a, b) => a - b)`.

**Q3. What's the difference between `slice` and `splice`?**
`slice(start, end)` is non-mutating — it returns a new array containing a portion of the original without changing it. `splice(start, deleteCount, ...items)` is mutating — it removes and/or inserts elements directly in the original array and returns the removed elements.

**Q4. How do you make a shallow copy of an array, and what's a common pitfall?**
Using the spread operator (`[...arr]`), `arr.slice()`, or `Array.from(arr)`. The pitfall is that these only copy one level deep — if the array contains objects or nested arrays, those inner references are shared between the copy and the original, so mutating a nested object affects both.

**Q5. What's the difference between `find` and `filter`?**
`find` returns the first single element that matches the predicate (or `undefined` if none match). `filter` returns a new array containing **all** matching elements (or an empty array if none match).

**Q6. How does `reduce` work? Give a practical example beyond summing numbers.**
`reduce(callback, initialValue)` iterates through the array, accumulating a single result via the callback `(accumulator, currentValue) => newAccumulator`. Example: counting word frequency — `words.reduce((acc, w) => ({ ...acc, [w]: (acc[w] || 0) + 1 }), {})`.

**Q7. What's the difference between `some` and `every`?**
`some` returns `true` if at least one element satisfies the predicate. `every` returns `true` only if all elements satisfy it. Both short-circuit as soon as the result is determined.

**Q8. How would you flatten a nested array?**
Using `Array.prototype.flat(depth)`, e.g., `[[1,2],[3,4]].flat()` returns `[1, 2, 3, 4]`. Pass `Infinity` as the depth to fully flatten deeply nested arrays.

**Q9. What's the difference between `Array.from()` and `Array.of()`?**
`Array.from()` creates an array from an iterable or array-like object (optionally applying a mapping function), e.g., `Array.from("abc")` → `['a','b','c']`. `Array.of()` creates an array from its given arguments directly, avoiding the single-numeric-argument quirk of `new Array(n)`.

**Q10. Why does `new Array(5)` behave differently from `Array.of(5)`?**
`new Array(5)` with a single numeric argument creates a sparse array with `length: 5` and no actual elements (`[ <5 empty items> ]`). `Array.of(5)` always creates an array containing the literal argument: `[5]`.

**Q11. How do you destructure an array while skipping certain elements?**
By leaving the corresponding position empty in the destructuring pattern, e.g., `const [first, , third] = [1, 2, 3];` skips index 1, giving `first = 1` and `third = 3`.

**Q12. What's the difference between the spread operator and rest parameters when used with arrays?**
Spread (`...`) expands an array into individual elements, used when merging arrays or passing elements as function arguments. Rest (`...`) does the opposite — it collects multiple individual elements into a single array, used in function parameters or destructuring, e.g., `const [first, ...rest] = arr`.
