**Asynchronous JavaScript** is what allows a single-threaded language to handle time-consuming operations — like network requests, timers, and file reads — without freezing the entire application. Understanding the event loop, promises, and `async`/`await` is essential for writing responsive, non-blocking JavaScript code, and is one of the most heavily tested topics in technical interviews.

---

### **Synchronous vs Asynchronous Execution**
JavaScript is **single-threaded**, meaning it can only execute one piece of code at a time, using a single **call stack**. In **synchronous** code, each statement waits for the previous one to finish before running. In **asynchronous** code, certain operations are handed off to be completed later, allowing the rest of the program to keep running without waiting.

```javascript
// Synchronous
console.log("First");
console.log("Second");
console.log("Third");
// Output:
// First
// Second
// Third

// Asynchronous
console.log("First");
setTimeout(() => console.log("Second (delayed)"), 0);
console.log("Third");
// Output:
// First
// Third
// Second (delayed)
```

Even with a `0`ms delay, `"Second (delayed)"` prints last — because `setTimeout` hands its callback off to be run **later**, after the currently executing synchronous code finishes.

---

### **The Event Loop**
JavaScript achieves asynchronous behavior through the **event loop**, which coordinates four main pieces:

1. **Call Stack**: Where currently executing function calls live, processed one at a time (LIFO — last in, first out).
2. **Web APIs / Node APIs**: Browser or Node-provided systems (like `setTimeout`, DOM events, or `fetch`) that handle async operations outside the main JavaScript thread.
3. **Callback Queue (Macrotask Queue)**: Holds callbacks from things like `setTimeout`, `setInterval`, and DOM events, waiting to be run once the call stack is empty.
4. **Microtask Queue**: Holds callbacks from Promises (`.then`/`.catch`/`.finally`) and `queueMicrotask`. This queue has **higher priority** than the macrotask queue.

**The event loop's job**: continuously check if the call stack is empty. If it is, first drain the **entire microtask queue**, then take just **one** task from the macrotask queue and repeat.

#### **Ordering Example**
```javascript
console.log("Start");

setTimeout(() => console.log("Timeout (macrotask)"), 0);

Promise.resolve().then(() => console.log("Promise (microtask)"));

console.log("End");

// Output:
// Start
// End
// Promise (microtask)
// Timeout (macrotask)
```

**Why this order?** `"Start"` and `"End"` run synchronously first. Then, before the event loop even looks at the macrotask (`setTimeout`) queue, it fully empties the microtask queue — so the Promise callback runs before the timeout callback, even though the timeout was scheduled first and has a `0ms` delay.

---

### **Callbacks and "Callback Hell"**
A **callback** is simply a function passed into another function to be executed later. When multiple asynchronous operations depend on each other, nesting callbacks leads to deeply indented, hard-to-read code often called **"callback hell"** or the **"pyramid of doom"**.

```javascript
getUser(1, function (user) {
  getPosts(user.id, function (posts) {
    getComments(posts[0].id, function (comments) {
      getReplies(comments[0].id, function (replies) {
        console.log(replies);
        // Deeply nested — hard to read, hard to handle errors, hard to maintain
      });
    });
  });
});
```

Problems with this pattern include difficult error handling (each level needs its own error check), poor readability, and awkward control flow if you need to run steps in parallel. Promises and `async`/`await` were introduced specifically to solve this.

---

### **Promises**
A **Promise** is an object representing the eventual completion (or failure) of an asynchronous operation. A promise exists in one of three **states**:

1. **Pending**: The initial state — neither fulfilled nor rejected.
2. **Fulfilled**: The operation completed successfully.
3. **Rejected**: The operation failed.

#### **Creating a Promise**
```javascript
const fetchData = new Promise((resolve, reject) => {
  const success = true;

  setTimeout(() => {
    if (success) {
      resolve("Data fetched successfully");
    } else {
      reject(new Error("Failed to fetch data"));
    }
  }, 1000);
});
```

#### **`.then()`, `.catch()`, and `.finally()`**
```javascript
fetchData
  .then((result) => {
    console.log(result); // Output (after 1s): Data fetched successfully
    return result.toUpperCase();
  })
  .then((upperResult) => {
    console.log(upperResult); // Output: DATA FETCHED SUCCESSFULLY
  })
  .catch((error) => {
    console.error(error.message); // Runs only if the promise was rejected
  })
  .finally(() => {
    console.log("Done, regardless of outcome");
  });
```

**Chaining** works because each `.then()` returns a **new promise**, allowing operations to be composed in sequence rather than nested inside each other — solving the callback hell problem.

---

### **Promise Combinators**
When you need to work with multiple promises at once, JavaScript provides four combinator methods:

| Method | Resolves when... | Rejects when... | Use case |
|---|---|---|---|
| **`Promise.all()`** | All promises fulfill | Any single promise rejects (fails fast) | Run several independent tasks, need all results |
| **`Promise.race()`** | The first promise settles (fulfilled or rejected) | Same — whichever settles first | Timeout patterns, "fastest response wins" |
| **`Promise.allSettled()`** | All promises settle (regardless of outcome) | Never rejects | Need results of every promise, even failures |
| **`Promise.any()`** | The first promise fulfills | Only if **all** promises reject | Need just one success, ignore failures |

```javascript
const p1 = Promise.resolve(1);
const p2 = new Promise((res) => setTimeout(() => res(2), 100));
const p3 = Promise.reject("Error in p3");

Promise.all([p1, p2]).then(console.log); // Output: [1, 2]

Promise.race([p1, p2]).then(console.log); // Output: 1 (settles first)

Promise.allSettled([p1, p3]).then(console.log);
// Output: [
//   { status: "fulfilled", value: 1 },
//   { status: "rejected", reason: "Error in p3" }
// ]

Promise.any([p3, p2]).then(console.log); // Output: 2 (first fulfilled, ignores p3's rejection)
```

---

### **`async`/`await`**
`async`/`await` is **syntactic sugar over Promises**, allowing asynchronous code to be written in a way that looks and reads like synchronous code. An `async` function always returns a Promise, and `await` pauses execution *within that function* until the awaited Promise settles.

```javascript
async function getWeather() {
  console.log("Fetching weather...");
  const result = await fetchData; // Pauses here until fetchData resolves
  console.log(result);
  return "Done";
}

getWeather().then((finalResult) => console.log(finalResult));
// Output:
// Fetching weather...
// (waits ~1s)
// Data fetched successfully
// Done
```

#### **Error Handling with `try`/`catch`**
Instead of chaining `.catch()`, errors from `await`ed promises are caught using a standard `try`/`catch` block:

```javascript
async function loadUserData() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    console.log(posts);
  } catch (error) {
    console.error("Something went wrong:", error.message);
  } finally {
    console.log("Request attempt finished");
  }
}
```

#### **Rewriting the Callback Hell Example**
```javascript
async function loadThread() {
  try {
    const user = await getUser(1);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    const replies = await getReplies(comments[0].id);
    console.log(replies);
  } catch (error) {
    console.error("Error loading thread:", error);
  }
}
```
This reads top-to-bottom like synchronous code, with a single, unified error-handling block — a dramatic readability improvement over nested callbacks.

---

### **The Fetch API**
The **Fetch API** provides a modern, promise-based way to make HTTP requests, replacing the older `XMLHttpRequest`.

#### **Using `.then()` Chains**
```javascript
fetch("https://jsonplaceholder.typicode.com/users/1")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    return response.json(); // .json() itself returns a promise
  })
  .then((data) => {
    console.log(data.name);
  })
  .catch((error) => {
    console.error("Fetch failed:", error.message);
  });
```

#### **Using `async`/`await`**
```javascript
async function getUser() {
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/users/1");
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const data = await response.json();
    console.log(data.name);
  } catch (error) {
    console.error("Fetch failed:", error.message);
  }
}

getUser();
```

Note that `fetch()` only rejects on network failure — a `404` or `500` response is still considered a "successful" fetch, which is why checking `response.ok` manually is necessary in both styles.

---

### **Best Practices**
- Prefer `async`/`await` over raw `.then()` chains for readability, especially with multiple sequential steps.
- Always handle errors — use `try`/`catch` with `async`/`await`, or `.catch()` with promise chains; never leave a promise chain without error handling.
- Use `Promise.all()` to run independent async operations concurrently instead of awaiting them one by one, which wastes time waiting unnecessarily.
- Remember that `fetch()` does not reject on HTTP error status codes — always check `response.ok` or `response.status`.
- Avoid mixing callback-based APIs with promises without wrapping them (`util.promisify` in Node, or manual `new Promise()` wrappers) for consistency.
- Never use `await` inside a loop when the operations are independent of each other — prefer `Promise.all()` with `.map()` to run them in parallel.

---

### **Interview Questions**

**Q1. What is the event loop, and why does JavaScript need it?**
The event loop is the mechanism that allows JavaScript's single-threaded call stack to handle asynchronous operations without blocking. It continuously checks whether the call stack is empty, and if so, moves queued callbacks (first microtasks, then one macrotask) onto the stack to be executed.

**Q2. What is the difference between the microtask queue and the macrotask (callback) queue?**
The microtask queue holds promise callbacks (`.then`/`.catch`/`.finally`) and `queueMicrotask` callbacks. The macrotask queue holds things like `setTimeout` and DOM event callbacks. The event loop always fully drains the microtask queue before processing even a single macrotask.

**Q3. Why does a `Promise.resolve().then()` callback run before a `setTimeout(fn, 0)` callback?**
Because promise callbacks go into the microtask queue, which the event loop always empties completely before picking up the next task from the macrotask queue (where `setTimeout` callbacks live), regardless of the timeout delay specified.

**Q4. What are the three states of a Promise?**
Pending (initial state, not yet settled), fulfilled (the operation succeeded, `resolve` was called), and rejected (the operation failed, `reject` was called). Once a promise settles (fulfilled or rejected), its state is permanent and cannot change again.

**Q5. What is "callback hell," and how do promises/async-await solve it?**
Callback hell is the deeply nested, hard-to-read code that results from chaining multiple asynchronous operations using nested callback functions. Promises solve it through flat `.then()` chaining, and `async`/`await` solves it further by letting asynchronous code be written with a linear, synchronous-looking structure.

**Q6. What is the difference between `Promise.all()` and `Promise.allSettled()`?**
`Promise.all()` rejects immediately if any one of the input promises rejects ("fail fast"), discarding the results of the others. `Promise.allSettled()` always waits for every promise to settle and never rejects, returning an array describing each promise's outcome (fulfilled or rejected).

**Q7. How does `async`/`await` relate to Promises?**
`async`/`await` is syntactic sugar built on top of Promises. An `async` function implicitly returns a Promise, and the `await` keyword pauses the function's execution until the awaited Promise settles, internally behaving like a `.then()` callback.

**Q8. How do you handle errors when using `async`/`await`?**
Wrap the `await` expressions in a `try`/`catch` block. Any rejected promise awaited inside the `try` block will throw, and execution jumps to the corresponding `catch` block, just like a synchronous exception.

**Q9. Does `fetch()` reject when the server responds with a 404 or 500 status code?**
No. `fetch()` only rejects the promise on network-level failures (like no internet connection or a DNS failure). An HTTP error response like 404 or 500 is still treated as a successfully completed fetch, so you must manually check `response.ok` or `response.status` to detect it.

**Q10. What is the difference between `Promise.race()` and `Promise.any()`?**
`Promise.race()` settles as soon as the first promise settles, whether it fulfills or rejects. `Promise.any()` settles as soon as the first promise fulfills, ignoring rejections — it only rejects if every single input promise rejects.

**Q11. How would you run three independent async operations in parallel and wait for all of them to finish?**
Start all three promises without awaiting them individually first, then pass them to `Promise.all()`:
```javascript
const [a, b, c] = await Promise.all([taskA(), taskB(), taskC()]);
```
This runs them concurrently rather than sequentially awaiting each one, which would waste time.

**Q12. What happens if you forget to `await` an async function call?**
The function still executes, but you get back the Promise object itself instead of its resolved value, and the calling code continues immediately without waiting for the async operation to complete — this is a common source of bugs like accessing `undefined` data or unhandled promise rejections.
