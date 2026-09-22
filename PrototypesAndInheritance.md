**Prototypes and Inheritance** form the foundation of how objects share behavior in JavaScript. Unlike classical object-oriented languages such as Java or C++ that rely purely on classes, JavaScript uses a **prototype-based** inheritance model, where objects inherit directly from other objects. Even the modern `class` syntax introduced in ES6 is just syntactic sugar built on top of this same prototype system, so understanding prototypes is essential to truly understanding JavaScript.

---

### **The Prototype Chain**
Every object in JavaScript has an internal, hidden link to another object called its **prototype**. This internal link is officially referred to as `[[Prototype]]` in the ECMAScript specification. When you try to access a property or method on an object and it isn't found directly on that object, JavaScript automatically looks up the chain to the object's prototype, then to *that* prototype's prototype, and so on — until it finds the property or reaches `null`, the end of the chain. This lookup path is called the **prototype chain**.

```javascript
const animal = {
  eats: true,
};

const rabbit = {
  jumps: true,
};

// Link rabbit's prototype to animal
Object.setPrototypeOf(rabbit, animal);

console.log(rabbit.jumps); // Output: true (own property)
console.log(rabbit.eats);  // Output: true (inherited via prototype chain)
```

#### **`__proto__` vs `Object.getPrototypeOf`**
`__proto__` is a legacy, non-standard (but widely supported) accessor property that exposes an object's `[[Prototype]]`. The modern, standardized way to read or set it is through `Object.getPrototypeOf()` and `Object.setPrototypeOf()`.

```javascript
console.log(rabbit.__proto__ === animal);              // Output: true
console.log(Object.getPrototypeOf(rabbit) === animal); // Output: true
```

| Feature | `__proto__` | `Object.getPrototypeOf` / `Object.setPrototypeOf` |
|---|---|---|
| **Standardization** | Legacy, added for web compatibility | Fully standardized in ES5/ES6 |
| **Usage** | `obj.__proto__` | `Object.getPrototypeOf(obj)` |
| **Recommended for production** | No | Yes |
| **Performance** | Can be slower in some engines | Generally optimized |

---

### **`prototype` Property vs the Actual Prototype Object**
This is a frequent source of confusion. Every **function** in JavaScript (when used as a constructor) has a special `prototype` property — this is **not** the function's own `[[Prototype]]`, but rather the object that will become the `[[Prototype]]` of any instance created with `new`.

```javascript
function Person(name) {
  this.name = name;
}

console.log(typeof Person.prototype); // Output: object

const alice = new Person("Alice");

console.log(Object.getPrototypeOf(alice) === Person.prototype);
// Output: true
```

So there are two related but distinct things:
1. **`Person.prototype`**: An object property that exists on the constructor function, used as the template for instances.
2. **`alice.[[Prototype]]`** (accessible via `Object.getPrototypeOf(alice)`): The actual internal link that `alice` uses to look up inherited properties, which happens to point to `Person.prototype`.

---

### **Constructor Functions and Prototype-Based Inheritance (Pre-ES6)**
Before ES6 classes existed, developers implemented inheritance using constructor functions and manually wiring up the prototype chain. Here's a full worked example: `Animal` as a base "class" and `Dog` inheriting from it.

```javascript
// Base constructor function
function Animal(name) {
  this.name = name;
}

// Add methods to the prototype so all instances share them (memory efficient)
Animal.prototype.eat = function () {
  console.log(`${this.name} is eating.`);
};

Animal.prototype.describe = function () {
  console.log(`I am ${this.name}, an animal.`);
};

// Dog constructor function
function Dog(name, breed) {
  Animal.call(this, name); // Call the parent constructor to set up `name`
  this.breed = breed;
}

// Set up inheritance: Dog.prototype's [[Prototype]] becomes Animal.prototype
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // Fix the constructor reference

// Add Dog-specific methods
Dog.prototype.bark = function () {
  console.log(`${this.name} says Woof!`);
};

const rex = new Dog("Rex", "Labrador");

rex.eat();      // Output: Rex is eating. (inherited from Animal.prototype)
rex.bark();     // Output: Rex says Woof! (own prototype method)
rex.describe(); // Output: I am Rex, an animal.

console.log(rex instanceof Dog);    // Output: true
console.log(rex instanceof Animal); // Output: true
```

**Why `Object.create(Animal.prototype)` and not `new Animal()`?**
Using `Object.create()` creates a new empty object whose prototype is `Animal.prototype`, without actually invoking the `Animal` constructor (which would run side effects and require arguments). This gives `Dog.prototype` the correct chain without any unwanted side effects.

---

### **ES6 `class` Syntax: Syntactic Sugar Over Prototypes**
ES6 introduced the `class` keyword, which makes prototype-based inheritance much easier to read and write. Under the hood, it still uses the exact same prototype chain mechanism.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  eat() {
    console.log(`${this.name} is eating.`);
  }

  describe() {
    console.log(`I am ${this.name}, an animal.`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Calls Animal's constructor
    this.breed = breed;
  }

  bark() {
    console.log(`${this.name} says Woof!`);
  }
}

const rex = new Dog("Rex", "Labrador");
rex.eat();  // Output: Rex is eating.
rex.bark(); // Output: Rex says Woof!

console.log(typeof Dog);                          // Output: function
console.log(Object.getPrototypeOf(Dog.prototype) === Animal.prototype);
// Output: true
```

This proves that `class`/`extends` is just sugar: `Dog` is still a function, and `Dog.prototype`'s `[[Prototype]]` still points to `Animal.prototype`, exactly as in the manual version above.

#### **`super`**
`super` is used in two ways inside a class:
1. **`super(...)`** inside a constructor: calls the parent class's constructor.
2. **`super.methodName()`** inside a method: calls the parent class's version of that method.

```javascript
class Cat extends Animal {
  describe() {
    super.describe(); // Call Animal's describe() first
    console.log("...and I am specifically a cat.");
  }
}

new Cat("Whiskers").describe();
// Output:
// I am Whiskers, an animal.
// ...and I am specifically a cat.
```

#### **Static Methods and Properties**
`static` members belong to the class itself, not to instances.

```javascript
class MathUtils {
  static PI = 3.14159;

  static square(n) {
    return n * n;
  }
}

console.log(MathUtils.PI);        // Output: 3.14159
console.log(MathUtils.square(4)); // Output: 16

const utils = new MathUtils();
console.log(utils.square);
// Output: undefined (static methods are not available on instances)
```

#### **Getters and Setters in Classes**
```javascript
class Circle {
  constructor(radius) {
    this._radius = radius;
  }

  get area() {
    return Math.PI * this._radius ** 2;
  }

  set radius(value) {
    if (value <= 0) throw new RangeError("Radius must be positive");
    this._radius = value;
  }
}

const circle = new Circle(5);
console.log(circle.area.toFixed(2)); // Output: 78.54
circle.radius = 10;
console.log(circle.area.toFixed(2)); // Output: 314.16
```

---

### **Multi-Level Inheritance**
Prototype chains can extend across multiple levels, exactly like a chain of classes.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  eat() {
    console.log(`${this.name} eats.`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`${this.name} barks.`);
  }
}

class Puppy extends Dog {
  play() {
    console.log(`${this.name} plays.`);
  }
}

const buddy = new Puppy("Buddy");
buddy.eat();  // Output: Buddy eats.   (from Animal)
buddy.bark(); // Output: Buddy barks.  (from Dog)
buddy.play(); // Output: Buddy plays.  (own method)

console.log(buddy instanceof Puppy);  // Output: true
console.log(buddy instanceof Dog);    // Output: true
console.log(buddy instanceof Animal); // Output: true
```

The prototype chain here looks like:
`buddy` → `Puppy.prototype` → `Dog.prototype` → `Animal.prototype` → `Object.prototype` → `null`

---

### **`instanceof` and the Prototype Chain**
The `instanceof` operator checks whether a constructor's `prototype` object appears **anywhere** in the target object's prototype chain.

```javascript
function checkInstance(obj, Constructor) {
  let proto = Object.getPrototypeOf(obj);
  while (proto !== null) {
    if (proto === Constructor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}

console.log(checkInstance(buddy, Dog));    // Output: true
console.log(buddy instanceof Dog);         // Output: true (same result, using the built-in operator)
```

This is exactly what `instanceof` does internally: it walks the chain looking for a match, rather than checking the object's direct class name.

---

### **Function Constructors vs Classes: Comparison**

| Feature | Constructor Function | ES6 Class |
|---|---|---|
| **Syntax** | Regular function + manual `prototype` wiring | `class` keyword with `extends`/`super` |
| **Hoisting** | Fully hoisted | Declaration hoisted but not initialized (TDZ) |
| **Calling without `new`** | Runs as a regular function (bugs likely) | Throws a `TypeError` |
| **Inheritance setup** | Manual (`Object.create`, fix `constructor`) | Automatic via `extends` |
| **Readability** | More verbose | Cleaner, more familiar to OOP developers |
| **Underlying mechanism** | Prototype chain | Prototype chain (identical under the hood) |

---

### **Best Practices**
- Prefer ES6 `class` syntax for readability, but understand that it compiles down to the same prototype mechanics.
- Put shared methods on the prototype (or in a class body, which does the same thing) rather than redefining them inside the constructor for every instance — this saves memory.
- Use `Object.create()` when you need a plain object with a specific prototype without invoking a constructor.
- Avoid modifying built-in prototypes (like `Array.prototype` or `Object.prototype`) in production code — it can cause unpredictable conflicts.
- Always call `super()` before using `this` in a derived class constructor; JavaScript enforces this and will throw a `ReferenceError` otherwise.
- Use `instanceof` for type checks (bearing in mind it can behave oddly across different realms/iframes), and prefer duck typing or `typeof` for simpler primitive checks.

---

### **Interview Questions**

**Q1. What is the prototype chain in JavaScript?**
It is the chain of internal `[[Prototype]]` links between objects. When a property isn't found on an object, JavaScript looks up the chain to the object's prototype, then that prototype's prototype, and so on, until it finds the property or reaches `null`.

**Q2. What is the difference between `__proto__` and `prototype`?**
`prototype` is a property that exists only on functions, used as the template for objects created with `new`. `__proto__` is an accessor on every object that exposes its actual internal `[[Prototype]]` link — the object it inherits from.

**Q3. How does `Object.create()` work, and why is it used when setting up inheritance manually?**
`Object.create(proto)` creates a brand-new object whose `[[Prototype]]` is set directly to `proto`, without invoking any constructor function. It's used in manual (pre-ES6) inheritance so that `Child.prototype` inherits from `Parent.prototype` without running `Parent`'s constructor and its side effects.

**Q4. Is ES6 `class` a completely new inheritance model in JavaScript?**
No. `class` is syntactic sugar over the existing prototype-based inheritance model. A class is still a function under the hood, and `extends` still wires up the prototype chain between `Child.prototype` and `Parent.prototype`.

**Q5. What does `super()` do inside a derived class's constructor?**
It calls the parent class's constructor, which is required to properly initialize `this` in the derived class. In JavaScript, `this` isn't available in a derived class constructor until `super()` has been called.

**Q6. How does `instanceof` actually work internally?**
`obj instanceof Constructor` walks up `obj`'s prototype chain checking whether `Constructor.prototype` appears anywhere in it. If found, it returns `true`; if the chain ends at `null` without a match, it returns `false`.

**Q7. What is the difference between static and instance methods in a class?**
Static methods/properties are defined on the class (constructor function) itself and are called directly on the class, not on instances. Instance methods are defined on the class's prototype and are accessible from every object created with `new`.
```javascript
class Foo {
  static bar() { return "static"; }
  baz() { return "instance"; }
}
Foo.bar();        // works
new Foo().baz();  // works
new Foo().bar();  // TypeError
```

**Q8. Why put methods on the prototype instead of inside the constructor?**
Methods defined inside a constructor (via `this.method = function(){}`) are recreated for every single instance, wasting memory. Methods defined on the prototype are created once and shared by reference across all instances.

**Q9. What happens if you forget to call `super()` in a derived class constructor?**
JavaScript throws a `ReferenceError` as soon as you try to access `this` (or implicitly at the end of the constructor), because `this` is not initialized in a derived class until the parent constructor runs via `super()`.

**Q10. How would you implement multi-level inheritance with constructor functions (without classes)?**
Chain `Object.create()` calls: set `Dog.prototype = Object.create(Animal.prototype)`, then `Puppy.prototype = Object.create(Dog.prototype)`, calling each parent constructor with `.call(this, ...)` inside the child constructor to initialize inherited properties.

**Q11. What is the difference between `Object.getPrototypeOf(obj)` and `obj.constructor.prototype`?**
They usually reference the same object, but `constructor.prototype` can be unreliable if the `constructor` property was reassigned or not fixed after manually replacing a `prototype` object. `Object.getPrototypeOf()` is the direct, standardized way to retrieve the true internal prototype link.

**Q12. Can you change an object's prototype after it has been created?**
Yes, using `Object.setPrototypeOf(obj, newProto)`, though this is discouraged in performance-sensitive code because it can de-optimize property lookups in most JavaScript engines. It's better to establish the correct prototype at creation time.
