**Objects** are collections of key-value pairs used to represent structured data and model real-world entities. Almost everything in JavaScript that isn't a primitive is an object under the hood, making objects one of the most fundamental building blocks of the language.

---

### **Object Literals**
The simplest and most common way to create an object.
```javascript
const person = {
  name: "Alice",
  age: 30,
  isEmployed: true,
};

console.log(person); // Output: { name: 'Alice', age: 30, isEmployed: true }
```

---

### **Property Access: Dot vs Bracket Notation**
```javascript
const car = { brand: "Toyota", model: "Corolla", "year of make": 2024 };

// Dot notation — clean, but requires a valid identifier key
console.log(car.brand); // Output: Toyota

// Bracket notation — required for dynamic keys or keys with special characters
console.log(car["year of make"]); // Output: 2024

const key = "model";
console.log(car[key]); // Output: Corolla (dot notation can't do this — car.key would look for a literal "key" property)
```

---

### **Computed Property Names**
Allows you to use an expression as a property key directly inside an object literal.
```javascript
const propName = "score";
const dynamicValue = 42;

const result = {
  [propName]: dynamicValue,
  [`${propName}_doubled`]: dynamicValue * 2,
};

console.log(result); // Output: { score: 42, score_doubled: 84 }
```

---

### **Shorthand Properties and Methods**
```javascript
const name = "Alice";
const age = 30;

// Shorthand property names (when variable name matches the key)
const person = { name, age };
console.log(person); // Output: { name: 'Alice', age: 30 }

// Shorthand method syntax
const calculator = {
  // Old way: add: function(a, b) { return a + b; }
  add(a, b) {
    return a + b;
  },
};
console.log(calculator.add(2, 3)); // Output: 5
```

---

### **`this` Inside Object Methods**
Inside a regular (non-arrow) method, `this` refers to the object the method was called **on** — the object to the left of the dot at call time.
```javascript
const user = {
  name: "Bob",
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  },
};

user.greet(); // Output: Hello, I'm Bob

const greetFn = user.greet;
// greetFn(); // Output: Hello, I'm undefined  (this` lost its binding when detached from `user`)
```
Arrow function methods do **not** get their own `this` — they inherit it from the enclosing lexical scope, which is usually not what you want for object methods:
```javascript
const user2 = {
  name: "Carol",
  greet: () => {
    console.log(`Hello, I'm ${this.name}`); // `this` here is NOT user2
  },
};
user2.greet(); // Output: Hello, I'm undefined
```
This is just the object-method basics — a full deep dive into how `this` is determined in every context (including `call`/`apply`/`bind` and classes) lives in `ThisKeyword.md`.

---

### **Object Constructors and Factory Functions**

#### **Constructor Function**
A regular function used with `new` to create multiple similar objects, following a class-like pattern (pre-dates ES6 `class` syntax, which is sugar over this).
```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.introduce = function () {
    console.log(`Hi, I'm ${this.name} and I'm ${this.age}.`);
  };
}

const p1 = new Person("Alice", 30);
const p2 = new Person("Bob", 25);

p1.introduce(); // Output: Hi, I'm Alice and I'm 30.
p2.introduce(); // Output: Hi, I'm Bob and I'm 25.
```

#### **Factory Function**
A plain function that returns a new object, without using `new` or `this` — avoids constructor pitfalls (like forgetting `new`).
```javascript
function createPerson(name, age) {
  return {
    name,
    age,
    introduce() {
      console.log(`Hi, I'm ${name} and I'm ${age}.`);
    },
  };
}

const p3 = createPerson("Carol", 28);
p3.introduce(); // Output: Hi, I'm Carol and I'm 28.
```

| Aspect | Constructor Function | Factory Function |
|---|---|---|
| Invocation | Requires `new` | Called normally, no `new` |
| Uses `this` | Yes | Usually no |
| Forgetting the keyword | `this` silently refers to global object/undefined (bug) | Not applicable — no `new` needed |
| Prototype sharing | Methods can be shared via `.prototype` | Methods are re-created per object unless manually shared |

---

### **`Object.freeze` vs `Object.seal`**
```javascript
const frozen = Object.freeze({ x: 1 });
frozen.x = 2;      // silently fails (throws in strict mode)
frozen.y = 3;       // can't add new properties
delete frozen.x;    // can't delete properties
console.log(frozen); // Output: { x: 1 } (completely unchanged)

const sealed = Object.seal({ x: 1 });
sealed.x = 2;       // ALLOWED — can still modify existing properties
sealed.y = 3;        // can't add new properties
delete sealed.x;     // can't delete properties
console.log(sealed); // Output: { x: 2 }
```

| Method | Add new properties | Modify existing properties | Delete properties |
|---|---|---|---|
| `Object.freeze()` | No | No | No |
| `Object.seal()` | No | **Yes** | No |
| (normal object) | Yes | Yes | Yes |

---

### **`Object.create`**
Creates a new object with a specified prototype, giving fine-grained control over inheritance without a constructor function.
```javascript
const animalPrototype = {
  speak() {
    console.log(`${this.name} makes a noise.`);
  },
};

const dog = Object.create(animalPrototype);
dog.name = "Rex";
dog.speak(); // Output: Rex makes a noise.

console.log(Object.getPrototypeOf(dog) === animalPrototype); // Output: true
```

---

### **`Object.assign`**
Copies enumerable own properties from one or more source objects into a target object — a common way to merge objects or create shallow copies.
```javascript
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3, a: 99 }; // later sources override earlier ones (and the target)

const merged = Object.assign(target, source1, source2);
console.log(merged); // Output: { a: 99, b: 2, c: 3 }
console.log(target);  // Output: { a: 99, b: 2, c: 3 } (target itself was mutated!)

// Safer merge that doesn't mutate an existing object
const safeMerged = Object.assign({}, source1, source2);
console.log(safeMerged); // Output: { b: 2, c: 3, a: 99 }
```

---

### **`Object.keys` / `Object.values` / `Object.entries`**
```javascript
const book = { title: "1984", author: "Orwell", year: 1949 };

console.log(Object.keys(book));   // Output: [ 'title', 'author', 'year' ]
console.log(Object.values(book)); // Output: [ '1984', 'Orwell', 1949 ]
console.log(Object.entries(book)); // Output: [ [ 'title', '1984' ], [ 'author', 'Orwell' ], [ 'year', 1949 ] ]

// entries() pairs perfectly with for...of and destructuring
for (const [key, value] of Object.entries(book)) {
  console.log(`${key}: ${value}`);
}
// Output:
// title: 1984
// author: Orwell
// year: 1949
```

---

### **Destructuring Objects**
```javascript
const employee = { name: "Dave", role: "Engineer", salary: 90000 };

// Basic destructuring
const { name, role } = employee;
console.log(name, role); // Output: Dave Engineer

// Renaming variables
const { name: employeeName, role: jobTitle } = employee;
console.log(employeeName, jobTitle); // Output: Dave Engineer

// Default values (used when the property is missing or undefined)
const { department = "Unassigned" } = employee;
console.log(department); // Output: Unassigned

// Renaming AND defaulting together
const { bonus: employeeBonus = 0 } = employee;
console.log(employeeBonus); // Output: 0

// Nested destructuring
const company = { name: "TechCorp", address: { city: "Austin", zip: "78701" } };
const { address: { city } } = company;
console.log(city); // Output: Austin
```

---

### **Spread Operator with Objects**
```javascript
const base = { a: 1, b: 2 };
const extra = { c: 3 };

const combined = { ...base, ...extra };
console.log(combined); // Output: { a: 1, b: 2, c: 3 }

// Overriding properties (later spread wins)
const updated = { ...base, b: 99 };
console.log(updated); // Output: { a: 1, b: 99 }

// Shallow copy
const copy = { ...base };
copy.a = 100;
console.log(base); // Output: { a: 1, b: 2 } (unaffected — top-level copy)
```

---

### **`instanceof` Operator**
Checks whether an object exists in the prototype chain of a given constructor — commonly used to verify an object's "type."
```javascript
function Car(make) {
  this.make = make;
}
const myCar = new Car("Honda");

console.log(myCar instanceof Car);    // Output: true
console.log(myCar instanceof Object); // Output: true (all objects inherit from Object)

console.log([] instanceof Array);  // Output: true
console.log([] instanceof Object); // Output: true

console.log("hello" instanceof String); // Output: false (primitives aren't instances)
console.log(new String("hello") instanceof String); // Output: true
```

---

### **Best Practices**
- Prefer object/array destructuring for cleaner, more readable function parameters and variable extraction.
- Use `Object.freeze()` for configuration objects or constants that should never change.
- Use factory functions or ES6 `class` syntax over raw constructor functions in modern code for safety and clarity.
- Avoid mutating objects directly when working with state in frameworks — use the spread operator or `Object.assign({}, ...)` to create new objects instead.
- Use `Object.entries()` with `for...of` when you need both keys and values while iterating.
- Remember that `Object.freeze`, spread, and `Object.assign` are all **shallow** — nested objects remain mutable unless deep-cloned.

---

### **Interview Questions**

**Q1. What's the difference between dot notation and bracket notation for accessing properties?**
Dot notation (`obj.key`) is concise but requires the key to be a valid identifier known at write-time. Bracket notation (`obj["key"]` or `obj[variable]`) is required when the key is dynamic, stored in a variable, or contains characters/spaces that aren't valid in an identifier.

**Q2. What's the difference between `Object.freeze()` and `Object.seal()`?**
`Object.freeze()` prevents adding, removing, AND modifying properties — the object becomes fully immutable at the top level. `Object.seal()` prevents adding or removing properties, but existing properties can still be reassigned.

**Q3. What does `Object.create()` do?**
It creates a new object with the specified object as its prototype, giving explicit control over the prototype chain without needing a constructor function or `class`.

**Q4. How does `this` behave in an object's regular method vs an arrow function method?**
In a regular method, `this` is determined dynamically by the object the method was called on (the receiver). In an arrow function assigned as a method, `this` is inherited lexically from the surrounding scope where the object literal was defined, not the object itself — so it typically doesn't refer to the object.

**Q5. What's the difference between a constructor function and a factory function?**
A constructor function is invoked with `new`, which creates a new object and binds `this` to it. A factory function is a plain function that explicitly builds and returns an object without `new` or reliance on `this`, avoiding bugs from a forgotten `new` keyword.

**Q6. What does `Object.assign()` do, and what's a common pitfall?**
It copies enumerable own properties from one or more source objects onto a target object, returning the mutated target. A common pitfall is passing an existing object as the target (mutating it unintentionally) instead of an empty object `{}` when the intent is just to merge/copy.

**Q7. How do `Object.keys()`, `Object.values()`, and `Object.entries()` differ?**
`Object.keys()` returns an array of an object's own enumerable property names. `Object.values()` returns an array of the corresponding values. `Object.entries()` returns an array of `[key, value]` pairs, useful for iterating with `for...of` and destructuring.

**Q8. How do you rename a variable while destructuring an object?**
Using the `originalKey: newName` syntax, e.g., `const { name: userName } = user;` extracts the `name` property into a variable called `userName`.

**Q9. Is the spread operator a deep copy or a shallow copy?**
Shallow. Spreading an object (`{ ...obj }`) copies top-level properties, but if a property's value is itself an object or array, both the original and the copy will reference the exact same nested object.

**Q10. What does `instanceof` actually check?**
It checks whether the constructor's `prototype` appears anywhere in the object's prototype chain (`Object.getPrototypeOf` chain), not the object's literal "type." It returns `false` for primitive values that haven't been wrapped in their object form.

**Q11. How would you merge two objects without mutating either of them?**
Using the spread operator into a new object literal: `const merged = { ...obj1, ...obj2 };`, or `Object.assign({}, obj1, obj2)` — both create a brand-new object rather than modifying an existing one.

**Q12. What happens if two spread sources (or `Object.assign` sources) have the same key?**
The property from the source that appears **last** wins, overwriting earlier values for that key — the same rule applies whether you're using `{ ...a, ...b }` or `Object.assign({}, a, b)`.
