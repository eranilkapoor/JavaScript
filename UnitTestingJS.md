**Unit testing** means writing small, automated pieces of code that verify individual functions or components behave correctly in isolation. Instead of manually clicking through an app to check that things still work after every change, unit tests let you run your entire test suite in seconds and catch regressions before they reach production.

---

### **Why Unit Testing Matters**
1. **Confidence to change code**: With good test coverage, you can refactor or add features without fear of silently breaking something else.
2. **Fast feedback**: Tests run in seconds, far faster than manually re-testing an application by hand.
3. **Living documentation**: A well-written test describes exactly how a function is expected to behave, which helps other developers (including future you) understand the code.
4. **Fewer regressions**: Bugs that are fixed and covered by a test are far less likely to silently reappear later.

#### **The Testing Pyramid (Briefly)**
Tests are usually grouped into three layers, from most numerous/fastest at the bottom to fewest/slowest at the top:

1. **Unit tests**: Test a single function or module in isolation, with dependencies mocked/stubbed. Fast, cheap, and should make up the bulk of your test suite.
2. **Integration tests**: Test how multiple units work together (e.g., a function that calls a database layer). Slower than unit tests, fewer in number.
3. **End-to-end (E2E) tests**: Test the entire application through the UI, simulating real user behavior (e.g., with Cypress or Playwright). Slowest and most brittle, so kept to a smaller number covering critical user flows.

The pyramid shape is a guideline: lots of fast unit tests at the base, a moderate number of integration tests in the middle, and a small number of high-value E2E tests at the top.

---

### **Mocha: The Test Runner**
**Mocha** is a flexible JavaScript test runner — it provides the structure to organize and run tests (`describe`/`it`), but it deliberately doesn't include an assertion library, so it's typically paired with one (like Chai).

#### **Installing Mocha**
```bash
npm install --save-dev mocha
```

#### **`describe` / `it` Structure**
- **`describe(name, fn)`**: Groups related tests together (a "test suite").
- **`it(name, fn)`**: Defines a single test case (a "spec").

```javascript
// test/math.test.js
const assert = require("assert");
const { add } = require("../math");

describe("Math utilities", () => {
  describe("add()", () => {
    it("adds two positive numbers", () => {
      assert.strictEqual(add(2, 3), 5);
    });

    it("handles negative numbers", () => {
      assert.strictEqual(add(-2, -3), -5);
    });
  });
});
```

Run it with:
```bash
npx mocha test/math.test.js
```
```
Math utilities
  add()
    ✔ adds two positive numbers
    ✔ handles negative numbers

2 passing (5ms)
```

Mocha ships with Node's built-in `assert` module out of the box, but most teams pair it with a richer assertion library like Chai for more expressive, readable assertions.

---

### **Chai: The Assertion Library**
**Chai** provides the assertion functions that check whether a value meets expectations, and it supports three different styles.

#### **Installing Chai**
```bash
npm install --save-dev chai
```

#### **The Three Styles**
```javascript
const chai = require("chai");
const expect = chai.expect;
const should = chai.should();
const assert = chai.assert;

// expect style — most popular, reads like a sentence
expect(5).to.equal(5);
expect([1, 2, 3]).to.include(2);

// should style — extends every object's prototype
(5).should.equal(5);
[1, 2, 3].should.include(2);

// assert style — closest to Node's built-in assert, classic TDD feel
assert.equal(5, 5);
assert.include([1, 2, 3], 2);
```

`expect` is the most commonly used style in modern codebases because it reads naturally and doesn't require modifying built-in prototypes the way `should` does.

#### **Common Assertions (`expect` style)**
```javascript
expect(4 + 1).to.equal(5);                       // strict equality (===)
expect({ a: 1 }).to.deep.equal({ a: 1 });         // deep object equality
expect("hello world").to.include("world");        // substring/array membership
expect([1, 2, 3]).to.have.lengthOf(3);             // length check
expect(() => { throw new Error("oops"); }).to.throw("oops"); // throws check
expect(null).to.be.null;                           // type/value checks
expect(5).to.be.a("number");                       // typeof check
```

Note the difference between `.equal()` (strict `===`, fails for objects/arrays unless they're the *same* reference) and `.deep.equal()` (compares structure/contents, not reference):
```javascript
expect({ a: 1 }).to.equal({ a: 1 });      // fails — different object references
expect({ a: 1 }).to.deep.equal({ a: 1 }); // passes — same structure
```

---

### **A Combined Mocha + Chai Example**
Testing a simple `add(a, b)` function end to end.

```javascript
// math.js
function add(a, b) {
  return a + b;
}

module.exports = { add };
```

```javascript
// test/math.test.js
const { expect } = require("chai");
const { add } = require("../math");

describe("add()", () => {
  it("returns the sum of two positive numbers", () => {
    expect(add(2, 3)).to.equal(5);
  });

  it("returns a negative number when both inputs are negative", () => {
    expect(add(-1, -1)).to.equal(-2);
  });

  it("returns NaN when given a non-numeric string", () => {
    expect(add(2, "x")).to.be.NaN;
  });
});
```

```
add()
  ✔ returns the sum of two positive numbers
  ✔ returns a negative number when both inputs are negative
  ✔ returns NaN when given a non-numeric string

3 passing (4ms)
```

---

### **Jest: An All-in-One Alternative**
**Jest** (built by Meta) bundles a test runner, assertion library, and mocking utilities into a single package — no need to install and wire up separate tools like Mocha + Chai + Sinon.

#### **Installing Jest**
```bash
npm install --save-dev jest
```

#### **The Same `add()` Example, in Jest**
```javascript
// math.test.js
const { add } = require("./math");

test("adds two positive numbers", () => {
  expect(add(2, 3)).toBe(5);
});

test("handles negative numbers", () => {
  expect(add(-2, -3)).toBe(-5);
});

// Jest also supports describe() for grouping, just like Mocha
describe("add()", () => {
  it("returns 0 when adding 0 and 0", () => {
    expect(add(0, 0)).toBe(0);
  });
});
```

Run it with:
```bash
npx jest
```

#### **Mocha + Chai vs Jest**
| | Mocha + Chai | Jest |
|---|---|---|
| **Setup** | Separate test runner + assertion library (+ optionally Sinon for mocking) | All-in-one — runner, assertions, and mocking included |
| **Assertion syntax** | `expect(x).to.equal(y)` | `expect(x).toBe(y)` |
| **Mocking** | Needs a separate library (Sinon) | Built in (`jest.fn()`, `jest.mock()`) |
| **Config** | More manual configuration | Sensible defaults, works out of the box |
| **Snapshot testing** | Not built in | Built in |
| **Popularity** | Long-standing, still common in Node.js libraries | Extremely popular, default choice for many React/frontend projects |

---

### **Testing Asynchronous Code**

#### **Returning a Promise**
If a test function returns a promise, both Mocha and Jest will wait for it to resolve (or fail the test if it rejects).
```javascript
// Mocha + Chai
it("resolves with user data", () => {
  return fetchUser(1).then((user) => {
    expect(user.id).to.equal(1);
  });
});
```

#### **`async`/`await` in Tests**
The cleanest, most common approach in modern code — just mark the test function `async`.
```javascript
// Works the same way in Mocha or Jest
it("resolves with user data", async () => {
  const user = await fetchUser(1);
  expect(user.id).to.equal(1);
});

// Jest equivalent
test("resolves with user data", async () => {
  const user = await fetchUser(1);
  expect(user.id).toBe(1);
});
```

#### **`done` Callback Style**
For older, callback-based asynchronous code (no promises), Mocha supports a `done` parameter — the test doesn't finish until `done()` is called.
```javascript
it("calls back with data", (done) => {
  fetchUserWithCallback(1, (err, user) => {
    if (err) return done(err);
    expect(user.id).to.equal(1);
    done(); // signals the test is complete
  });
});
```
Forgetting to call `done()` (or to return a promise) is a common source of tests that silently "hang" or time out.

---

### **Mocking and Spies**
Real unit tests often need to isolate the function under test from its dependencies (network calls, timers, other modules) using **mocks** and **spies**.

#### **With Sinon.js (commonly paired with Mocha)**
```javascript
const sinon = require("sinon");

const logger = { log: () => {} };
const spy = sinon.spy(logger, "log");

logger.log("hello");

console.log(spy.calledOnce); // Output: true
console.log(spy.calledWith("hello")); // Output: true
```

#### **With Jest's Built-In `jest.fn()`**
```javascript
const mockCallback = jest.fn();

[1, 2, 3].forEach(mockCallback);

expect(mockCallback).toHaveBeenCalledTimes(3);
expect(mockCallback).toHaveBeenCalledWith(1, 0, [1, 2, 3]);
```

A **spy** wraps a real function to record how it was called (while still calling the original), while a **mock/stub** replaces a function entirely with fake, controllable behavior — both are essential for testing code in isolation from slow or unpredictable dependencies like network requests.

---

### **Best Practices**
- Keep unit tests small and focused — one behavior per `it()`/`test()` block, with a clear, descriptive name.
- Follow the Arrange-Act-Assert pattern: set up inputs, call the function, then assert the result.
- Don't test implementation details — test the observable behavior (inputs/outputs), so tests survive internal refactors.
- Mock external dependencies (network, database, timers) in unit tests so they run fast and deterministically; save real integrations for integration tests.
- Aim for meaningful coverage of edge cases and error paths, not just the happy path — and don't chase 100% coverage as a goal in itself.
- Run tests automatically in CI on every push/PR so regressions are caught before merging.

---

### **Interview Questions**

**Q1. What is unit testing, and why is it valuable?**
Unit testing is writing automated tests that verify individual functions or components work correctly in isolation. It gives fast feedback on regressions, documents expected behavior, and lets developers refactor or extend code with confidence.

**Q2. What is the testing pyramid?**
A model describing the recommended mix of test types: many fast, cheap unit tests at the base; fewer integration tests in the middle testing how units work together; and a small number of slow, high-value end-to-end tests at the top simulating real user flows.

**Q3. What's the difference between Mocha and Chai?**
Mocha is a test runner — it provides `describe`/`it` structure and executes tests, but has no built-in assertion library. Chai is an assertion library that provides the `expect`/`should`/`assert` syntax used to check values inside those tests. They're commonly used together.

**Q4. What are the three assertion styles Chai supports?**
`expect` (e.g., `expect(x).to.equal(5)`), `should` (e.g., `x.should.equal(5)`, which extends `Object.prototype`), and `assert` (e.g., `assert.equal(x, 5)`, closest to Node's built-in `assert`). `expect` is the most commonly used in modern projects.

**Q5. What's the difference between `.equal()` and `.deep.equal()` in Chai?**
`.equal()` checks strict equality (`===`), so two different object/array instances with identical contents will fail. `.deep.equal()` recursively compares structure and values, so two separately-created objects with the same shape will pass.

**Q6. How does Jest differ from Mocha + Chai?**
Jest is an all-in-one framework bundling a test runner, assertion library (`expect`), and mocking utilities (`jest.fn()`, `jest.mock()`) in one package with minimal setup. Mocha and Chai are separate libraries you combine yourself, often adding a third tool (like Sinon) for mocking.

**Q7. How do you test a function that returns a promise?**
Either return the promise from the test (the framework waits for it to resolve/reject), or mark the test function `async` and `await` the result before asserting:
```javascript
it("works", async () => {
  const result = await someAsyncFn();
  expect(result).to.equal(expected);
});
```

**Q8. What is the `done` callback used for in Mocha?**
It's used to signal that an asynchronous test (typically one using callback-style APIs rather than promises) has finished. The test framework waits until `done()` is called (or times out), which is necessary because Mocha can't otherwise know when callback-based async code has completed.

**Q9. What's the difference between a spy and a mock/stub?**
A spy wraps a real function to record how/when it was called while still executing the original behavior. A mock (or stub) replaces the function entirely with fake, controllable behavior, letting you simulate specific return values, errors, or edge cases without invoking the real implementation.

**Q10. Why would you mock a network call in a unit test instead of hitting a real API?**
Real network calls make tests slow, flaky (dependent on network conditions and external service uptime), and hard to control for specific scenarios (like simulating an error response). Mocking keeps unit tests fast, deterministic, and able to test edge cases (like a 500 error) on demand.

**Q11. What does `toHaveBeenCalledWith()` check in Jest?**
It's an assertion on a `jest.fn()` mock function verifying it was called with specific arguments at least once, which is useful for confirming that a function under test correctly calls its dependencies with the right data.

**Q12. Should unit tests cover implementation details or observable behavior?**
Observable behavior — inputs and outputs (or side effects), not internal implementation. Testing implementation details makes tests brittle, breaking on harmless internal refactors even when the function's actual behavior hasn't changed.
