**Event Handling** is how JavaScript responds to user interactions — clicks, key presses, form submissions, mouse movements, and more. A solid understanding of how events are registered, how they propagate through the DOM, and how to efficiently manage many event-driven elements is essential for building interactive, performant web applications.

---

### **Registering Events**
There are three common ways to attach an event handler to an element.

1. **Inline HTML handlers**: Written directly in the HTML attribute. Mixes markup with logic and is generally discouraged.
   ```html
   <button onclick="handleClick()">Click me</button>
   ```

2. **On-property handlers**: Assigned directly to a DOM element's property in JavaScript. Only **one** handler can be assigned per event type — assigning a second overwrites the first.
   ```javascript
   const button = document.querySelector("button");
   button.onclick = function () {
     console.log("Clicked!");
   };
   ```

3. **`addEventListener`**: The modern, preferred approach. Supports **multiple** handlers on the same event, fine-grained control (capturing/bubbling, options), and easy removal.
   ```javascript
   button.addEventListener("click", function () {
     console.log("Clicked via addEventListener");
   });

   button.addEventListener("click", function () {
     console.log("A second handler, also runs");
   });
   // Output (on click):
   // Clicked via addEventListener
   // A second handler, also runs
   ```

#### **Why `addEventListener` is Preferred**

| Feature | Inline HTML | On-property (`onclick`) | `addEventListener` |
|---|---|---|---|
| **Multiple handlers per event** | No | No (overwrites) | Yes |
| **Separation of concerns (HTML/JS)** | No | Partial | Yes |
| **Control over capturing/bubbling** | No | No | Yes |
| **Easy removal** | No | Overwrite with `null` | Yes (`removeEventListener`) |
| **Supports options (`once`, `passive`)** | No | No | Yes |

---

### **The Event Object**
Every event handler receives an **event object** describing what happened.

```javascript
button.addEventListener("click", function (event) {
  console.log(event.type);           // Output: "click"
  console.log(event.target);         // The actual element that triggered the event
  console.log(event.currentTarget);  // The element the listener is attached to
});
```

1. **`target`**: The specific element that originally triggered the event (e.g., the exact `<li>` you clicked inside a list).
2. **`currentTarget`**: The element the event listener is actually attached to (relevant during bubbling — `target` and `currentTarget` can differ).
3. **`type`**: The name of the event (e.g., `"click"`, `"submit"`).
4. **`preventDefault()`**: Stops the browser's default behavior for that event (e.g., stopping a form from submitting or a link from navigating).
5. **`stopPropagation()`**: Stops the event from continuing to bubble (or capture) to other elements.

```javascript
const form = document.querySelector("form");

form.addEventListener("submit", function (event) {
  event.preventDefault(); // Prevent the page from reloading
  console.log("Form submission intercepted");
});
```

---

### **Event Propagation: Capturing vs Bubbling**
When an event occurs on a nested element, it doesn't just fire on that element — it travels through the DOM tree in two phases:

1. **Capturing phase**: The event travels **down** from `window`/`document` to the target element.
2. **Target phase**: The event reaches the actual element that was interacted with.
3. **Bubbling phase**: The event travels back **up** from the target element to `window`/`document`.

By default, `addEventListener` listens during the **bubbling** phase. Passing `true` (or `{ capture: true }`) as the third argument makes it listen during the **capturing** phase instead.

```html
<div id="outer">
  Outer
  <div id="middle">
    Middle
    <div id="inner">Inner</div>
  </div>
</div>
```

```javascript
const outer = document.getElementById("outer");
const middle = document.getElementById("middle");
const inner = document.getElementById("inner");

outer.addEventListener("click", () => console.log("Outer - Bubbling"));
middle.addEventListener("click", () => console.log("Middle - Bubbling"));
inner.addEventListener("click", () => console.log("Inner - Bubbling"));

outer.addEventListener("click", () => console.log("Outer - Capturing"), true);
middle.addEventListener("click", () => console.log("Middle - Capturing"), true);
inner.addEventListener("click", () => console.log("Inner - Capturing"), true);

// Clicking on the innermost "Inner" div outputs:
// Outer - Capturing
// Middle - Capturing
// Inner - Capturing
// Inner - Bubbling
// Middle - Bubbling
// Outer - Bubbling
```

**Order explained**: The event first travels down (capturing: Outer → Middle → Inner), fires on the actual target, and then travels back up (bubbling: Inner → Middle → Outer).

---

### **Event Delegation**
**Event delegation** is a pattern where, instead of attaching a separate event listener to every individual child element, you attach a **single listener to a common parent** and use `event.target` to determine which child was actually interacted with. This works because events bubble up from the child to the parent.

#### **Why It's Useful**
- **Performance**: One listener instead of hundreds/thousands, especially in large lists.
- **Dynamic elements**: Automatically works for elements added to the DOM later, without needing to re-attach listeners.

#### **Worked Example: A List with a Single Listener**
```html
<ul id="task-list">
  <li>Buy milk</li>
  <li>Walk the dog</li>
  <li>Write code</li>
</ul>
<button id="add-task">Add Task</button>
```

```javascript
const list = document.getElementById("task-list");
const addButton = document.getElementById("add-task");

// A single listener on the parent handles clicks on ANY <li>, present or future
list.addEventListener("click", function (event) {
  if (event.target.tagName === "LI") {
    event.target.classList.toggle("completed");
    console.log(`Toggled: ${event.target.textContent}`);
  }
});

// New items added later are automatically handled — no new listener needed
addButton.addEventListener("click", function () {
  const newItem = document.createElement("li");
  newItem.textContent = "New dynamic task";
  list.appendChild(newItem);
});
```

Without delegation, you would need to attach a listener to every `<li>` individually, and remember to attach a new one every time a task is added — event delegation eliminates that entirely.

---

### **Common Events**

| Event | Fires when... |
|---|---|
| **`click`** | An element is clicked |
| **`submit`** | A form is submitted |
| **`input`** | The value of an `<input>`/`<textarea>` changes (fires on every keystroke) |
| **`change`** | An element's value is committed (e.g., losing focus on a text input, or selecting a `<select>` option) |
| **`keydown`** / **`keyup`** | A key is pressed down / released |
| **`mouseover`** / **`mouseout`** | The pointer enters / leaves an element (bubbles, fires for child transitions too) |
| **`load`** | A resource (image, script, or the whole page) finishes loading |
| **`DOMContentLoaded`** | The initial HTML document has been completely parsed, without waiting for stylesheets/images |

```javascript
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM is fully parsed and ready");
});

window.addEventListener("load", () => {
  console.log("All resources (images, styles) finished loading");
});

const input = document.querySelector("input");
input.addEventListener("input", (e) => console.log("Typing:", e.target.value));
input.addEventListener("change", (e) => console.log("Committed:", e.target.value));
```

---

### **Removing Event Listeners**
To remove a listener, you must pass the **exact same function reference** used when it was added — anonymous inline functions cannot be removed.

```javascript
function handleClick() {
  console.log("Clicked!");
}

button.addEventListener("click", handleClick);
button.removeEventListener("click", handleClick); // Successfully removes it

button.addEventListener("click", () => console.log("Oops"));
button.removeEventListener("click", () => console.log("Oops"));
// Does NOT remove it — this is a different function reference
```

#### **The `once` and `passive` Options**
`addEventListener` accepts an options object as its third argument:

```javascript
// `once`: automatically removes the listener after it fires a single time
button.addEventListener(
  "click",
  () => console.log("This only runs once"),
  { once: true }
);

// `passive`: tells the browser the listener will never call preventDefault(),
// allowing the browser to optimize scroll performance (commonly used for touch/scroll events)
document.addEventListener(
  "scroll",
  () => console.log("Scrolling..."),
  { passive: true }
);
```

| Option | Purpose |
|---|---|
| **`once`** | Automatically removes the listener after it fires one time |
| **`passive`** | Promises the browser `preventDefault()` won't be called, improving scroll/touch performance |
| **`capture`** | Registers the listener for the capturing phase instead of bubbling |

---

### **Best Practices**
- Prefer `addEventListener` over inline handlers or on-property assignment for maintainability and multiple-handler support.
- Use event delegation for lists or containers with many similar children, especially if items are added/removed dynamically.
- Always call `event.preventDefault()` when you need to override default browser behavior (e.g., custom form validation).
- Use `{ passive: true }` on scroll/touch listeners that don't call `preventDefault()`, to keep scrolling smooth.
- Remove listeners you no longer need (e.g., in cleanup logic for components or single-page app route changes) to avoid memory leaks.
- Use `stopPropagation()` sparingly — it can break other legitimate listeners relying on bubbling, including analytics or delegated handlers higher up the tree.

---

### **Interview Questions**

**Q1. What is the difference between `addEventListener` and setting an `onclick` property?**
`onclick` allows only a single handler per event — assigning a new one overwrites the previous. `addEventListener` allows multiple handlers on the same event and type, and provides more control such as capturing/bubbling and options like `once` or `passive`.

**Q2. What is the difference between event capturing and event bubbling?**
Capturing is the phase where the event travels from the root of the document down to the target element. Bubbling is the phase where, after reaching the target, the event travels back up from the target to the root. By default, listeners registered with `addEventListener` fire during the bubbling phase.

**Q3. What is the difference between `event.target` and `event.currentTarget`?**
`event.target` is the actual element that triggered the event (e.g., the specific `<li>` clicked). `event.currentTarget` is the element the listener is attached to, which can be a different, higher-level ancestor element when using event delegation.

**Q4. What is event delegation, and why is it useful?**
Event delegation attaches a single listener to a common parent element instead of individual listeners on each child, relying on event bubbling and `event.target` to determine which child was interacted with. It improves performance with many elements and automatically works for dynamically added children without needing new listeners.

**Q5. How do you stop a form from submitting normally when using JavaScript to validate it?**
Call `event.preventDefault()` inside the `submit` event handler, which cancels the browser's default action of submitting the form and reloading the page.

**Q6. What is the difference between `stopPropagation()` and `preventDefault()`?**
`stopPropagation()` stops the event from continuing to bubble (or capture) to other elements in the DOM tree, but does not stop the browser's default behavior. `preventDefault()` stops the browser's default action (like navigating a link or submitting a form), but does not stop the event from propagating to other listeners.

**Q7. Why can't you remove an event listener added as an anonymous arrow function?**
`removeEventListener` requires a reference to the exact same function that was passed to `addEventListener`. An anonymous function created inline creates a new function reference every time, so there's no way to reference it later for removal — the listener must be stored in a named variable or function declaration.

**Q8. What does the `once` option do in `addEventListener`?**
It automatically removes the event listener after it has been invoked a single time, useful for one-time interactions like dismissing a welcome modal without manually calling `removeEventListener`.

**Q9. What is the difference between the `input` and `change` events on a text field?**
`input` fires immediately on every value change, including every keystroke. `change` fires only when the value is "committed," typically when the element loses focus after its value has changed (or immediately for elements like `<select>` and checkboxes).

**Q10. What is the purpose of the `passive` option in `addEventListener`?**
It tells the browser in advance that the listener will never call `preventDefault()`, allowing the browser to start scrolling immediately without waiting for the listener to finish executing — this significantly improves scroll performance, especially for touch and wheel events.

**Q11. What is the difference between `DOMContentLoaded` and the `load` event on `window`?**
`DOMContentLoaded` fires as soon as the HTML has been fully parsed into the DOM, without waiting for images, stylesheets, or other external resources. `load` fires later, only after all of those resources have fully finished loading.

**Q12. How would you attach a click handler to 1,000 buttons efficiently?**
Instead of attaching 1,000 individual listeners, use event delegation: attach a single listener to their common parent container, and inside the handler check `event.target` (e.g., via `closest()` or a class check) to determine which button was actually clicked.
