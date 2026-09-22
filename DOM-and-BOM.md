**The DOM (Document Object Model)** and **BOM (Browser Object Model)** are the two APIs that let JavaScript interact with a web page and the browser itself. The DOM represents the HTML document as a tree of objects that can be read and manipulated, while the BOM gives JavaScript access to browser-level features like the URL, navigation history, and screen information. Together, they are what allow JavaScript to turn a static HTML page into an interactive application.

---

### **What is the DOM?**
The **DOM** is a tree-structured, in-memory representation of an HTML document. Every HTML tag becomes a **node** in this tree, and JavaScript can read, add, remove, or modify these nodes to change what the user sees, without reloading the page.

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="container">
      <p>Hello, DOM!</p>
    </div>
  </body>
</html>
```

This HTML is represented internally roughly as:

```
document
 └─ html
     └─ body
         └─ div#container
             └─ p
                 └─ "Hello, DOM!" (text node)
```

```javascript
console.log(document.body.tagName);
// Output: BODY
```

---

### **Selecting Elements**
JavaScript provides several methods to find elements in the DOM tree.

1. **`getElementById`**: Selects a single element by its `id` attribute. Fastest and most specific.
   ```javascript
   const container = document.getElementById("container");
   ```

2. **`getElementsByClassName`**: Selects all elements with a given class name. Returns a **live HTMLCollection**.
   ```javascript
   const items = document.getElementsByClassName("item");
   ```

3. **`getElementsByTagName`**: Selects all elements with a given tag name. Also returns a **live HTMLCollection**.
   ```javascript
   const paragraphs = document.getElementsByTagName("p");
   ```

4. **`querySelector`**: Selects the **first** element matching any CSS selector.
   ```javascript
   const firstItem = document.querySelector(".item");
   const byAttr = document.querySelector("input[type='text']");
   ```

5. **`querySelectorAll`**: Selects **all** elements matching a CSS selector. Returns a **static NodeList**.
   ```javascript
   const allItems = document.querySelectorAll(".item");
   allItems.forEach((item) => console.log(item.textContent));
   ```

#### **NodeList vs Live HTMLCollection**
This is a common source of bugs: `getElementsBy*` methods return **live** collections that automatically update as the DOM changes, while `querySelectorAll` returns a **static** snapshot taken at the moment of the call.

```javascript
const liveDivs = document.getElementsByTagName("div"); // live HTMLCollection
const staticDivs = document.querySelectorAll("div");    // static NodeList

const newDiv = document.createElement("div");
document.body.appendChild(newDiv);

console.log(liveDivs.length);   // Output: increased by 1 (auto-updated)
console.log(staticDivs.length); // Output: unchanged (snapshot from before)
```

| Feature | `HTMLCollection` (`getElementsBy*`) | `NodeList` (`querySelectorAll`) |
|---|---|---|
| **Live or Static** | Live — updates automatically | Static — fixed snapshot |
| **Iteration** | No `forEach` (must convert to array) | Has `forEach` directly |
| **Selector power** | Limited (id, class, tag name) | Full CSS selector support |
| **Return for single match** | Collection | Collection (use `querySelector` for one element) |

---

### **Traversing the DOM**
Once you have a reference to an element, you can navigate to related nodes:

```javascript
const item = document.querySelector(".item");

console.log(item.parentNode);         // The direct parent node
console.log(item.parentElement);      // The direct parent element (like parentNode, but always an Element)
console.log(item.children);           // Live HTMLCollection of child elements (ignores text nodes)
console.log(item.childNodes);         // NodeList including text/comment nodes
console.log(item.firstElementChild);  // First child element
console.log(item.lastElementChild);   // Last child element
console.log(item.nextElementSibling); // Next sibling element
console.log(item.previousElementSibling); // Previous sibling element
```

```javascript
// Example: highlight every sibling after the first .item element
const first = document.querySelector(".item");
let sibling = first.nextElementSibling;

while (sibling) {
  sibling.style.color = "red";
  sibling = sibling.nextElementSibling;
}
```

---

### **Modifying the DOM**

#### **Creating and Inserting Elements**
```javascript
const list = document.getElementById("list");

// Create a new element
const newItem = document.createElement("li");
newItem.textContent = "New Item";

// Append it as the last child
list.appendChild(newItem);

// Insert before a specific existing element
const referenceItem = document.querySelector("li.first");
list.insertBefore(newItem, referenceItem);

// Modern alternatives
list.append(newItem);          // Can append multiple nodes/strings
list.prepend(newItem);         // Insert at the beginning
referenceItem.before(newItem); // Insert immediately before referenceItem
referenceItem.after(newItem);  // Insert immediately after referenceItem
```

#### **Removing and Replacing Elements**
```javascript
const oldItem = document.querySelector("li.old");

// Remove a child (called on the parent)
list.removeChild(oldItem);

// Modern alternative: remove a node directly
oldItem.remove();

// Replace one child with another
const replacement = document.createElement("li");
replacement.textContent = "Replacement";
list.replaceChild(replacement, oldItem);
```

#### **`innerHTML` vs `textContent` vs `innerText`**

```javascript
const box = document.getElementById("box");

box.innerHTML = "<strong>Bold</strong> text"; // Parses and renders HTML tags
box.textContent = "<strong>Bold</strong> text"; // Inserted as literal, escaped text
box.innerText = "Visible text only";            // Respects CSS visibility/rendering
```

| Property | Parses HTML? | Includes hidden text? | Performance | Security |
|---|---|---|---|---|
| **`innerHTML`** | Yes | Yes | Slower (triggers re-parsing) | Risk of XSS with untrusted input |
| **`textContent`** | No (treated as plain text) | Yes | Fastest | Safe |
| **`innerText`** | No | No (only rendered/visible text) | Slower (triggers reflow) | Safe |

---

### **Modifying Attributes and Classes**
```javascript
const link = document.querySelector("a");

// Attributes
link.setAttribute("href", "https://example.com");
link.getAttribute("href");        // Output: "https://example.com"
link.removeAttribute("target");
link.hasAttribute("href");        // Output: true

// Classes
link.classList.add("active");
link.classList.remove("disabled");
link.classList.toggle("highlight");  // Adds if absent, removes if present
link.classList.contains("active");   // Output: true
link.className = "btn btn-primary";  // Overwrites all classes at once
```

---

### **The BOM (Browser Object Model)**
While the DOM represents the *document*, the **BOM** represents the *browser* itself. The global `window` object is the root of the BOM and also acts as the global object in browser JavaScript.

1. **`window`**: The global object representing the browser tab/window. All global variables and functions become properties of `window`. It also provides methods like `alert()`, `setTimeout()`, and `open()`.
   ```javascript
   console.log(window.innerWidth, window.innerHeight); // Viewport size
   window.alert("Hello!");
   ```

2. **`navigator`**: Provides information about the browser and device.
   ```javascript
   console.log(navigator.userAgent); // Browser/OS details
   console.log(navigator.language);  // e.g., "en-US"
   console.log(navigator.onLine);    // true/false network status
   ```

3. **`location`**: Provides information about, and control over, the current URL.
   ```javascript
   console.log(location.href);     // Full URL
   console.log(location.hostname); // e.g., "example.com"
   console.log(location.pathname); // e.g., "/products"
   location.reload();              // Reloads the page
   location.assign("https://example.com"); // Navigates to a new URL
   ```

4. **`history`**: Allows interaction with the browser's session history.
   ```javascript
   history.back();    // Go to the previous page
   history.forward();  // Go to the next page
   history.pushState({}, "", "/new-path"); // Change URL without reloading
   ```

5. **`screen`**: Provides information about the user's physical screen.
   ```javascript
   console.log(screen.width, screen.height); // Full screen resolution
   console.log(screen.availWidth, screen.availHeight); // Usable area (minus taskbars, etc.)
   ```

#### **DOM vs BOM**

| Aspect | DOM | BOM |
|---|---|---|
| **Represents** | The HTML document structure | The browser window/environment |
| **Root object** | `document` | `window` |
| **Standardized** | Yes, by W3C/WHATWG | Partially — largely a de facto standard |
| **Examples** | `getElementById`, `createElement` | `location`, `navigator`, `history`, `screen` |

---

### **Best Practices**
- Prefer `querySelector`/`querySelectorAll` for their flexible CSS-selector syntax, but be aware they return static snapshots.
- Use `textContent` instead of `innerHTML` when inserting plain text to avoid XSS vulnerabilities.
- Cache DOM references in variables rather than repeatedly querying the DOM inside loops — DOM queries are relatively expensive.
- Use `classList` methods instead of manipulating `className` strings directly for cleaner, less error-prone class management.
- Batch DOM updates where possible (e.g., build elements off-DOM, then append once) to minimize layout thrashing/reflows.
- Use `DocumentFragment` when inserting many elements at once to avoid triggering multiple reflows.

---

### **Interview Questions**

**Q1. What is the DOM?**
The DOM (Document Object Model) is a tree-structured, programmatic representation of an HTML document that JavaScript can read and manipulate. Each HTML element becomes a node in this tree, allowing scripts to dynamically change content, structure, and styling.

**Q2. What is the difference between `getElementById` and `querySelector`?**
`getElementById` selects a single element strictly by its `id` attribute and is generally the fastest option. `querySelector` accepts any valid CSS selector and returns the first matching element, offering much greater flexibility at a slight performance cost.

**Q3. What is the difference between a live HTMLCollection and a static NodeList?**
A live `HTMLCollection` (returned by `getElementsByClassName`/`getElementsByTagName`) automatically reflects DOM changes made after it was retrieved. A static `NodeList` (returned by `querySelectorAll`) is a fixed snapshot that does not update even if matching elements are later added or removed.

**Q4. What is the difference between `innerHTML`, `textContent`, and `innerText`?**
`innerHTML` parses and renders its string as HTML markup. `textContent` inserts the string as literal, unparsed text and includes hidden elements' text. `innerText` behaves similarly to `textContent` but respects CSS styling, only returning visibly rendered text and triggering a reflow to compute it.

**Q5. Why is `textContent` generally safer than `innerHTML`?**
`innerHTML` parses its input as HTML, so inserting untrusted user input can lead to Cross-Site Scripting (XSS) attacks if the input contains malicious `<script>` tags or event handler attributes. `textContent` always treats input as plain text, neutralizing that risk.

**Q6. What is the BOM, and how is it different from the DOM?**
The BOM (Browser Object Model) represents the browser environment itself — things like the URL, history, and screen — rooted at the `window` object. The DOM represents only the HTML document's structure, rooted at `document`. The DOM is a W3C standard; the BOM is mostly a de facto standard shaped by browser vendors.

**Q7. How do you remove an element from the DOM?**
Either call `parentNode.removeChild(element)`, or use the more modern and direct `element.remove()` method, which removes the element from its parent without needing a reference to the parent.

**Q8. What is the difference between `parentNode` and `parentElement`?**
`parentNode` returns the direct parent node, which could be an element, a document, or a document fragment. `parentElement` returns the parent specifically as an `Element`, or `null` if the parent isn't an element (e.g., when the parent is the `document` node itself).

**Q9. How would you efficiently insert 1,000 new list items into the DOM?**
Build the items inside a `DocumentFragment` (or as a string via `innerHTML`/template building) first, then append the fragment to the DOM in a single operation. This avoids triggering a reflow/repaint for every individual insertion.

**Q10. What does `location.href` vs `location.assign()` do, and what's the difference?**
Both navigate the browser to a new URL. Setting `location.href = url` and calling `location.assign(url)` behave almost identically — both add a new entry to browser history. `location.replace(url)` is different in that it replaces the current history entry instead of adding a new one.

**Q11. How do you check if an element has a specific class without using `classList`?**
You can check the `className` string directly, e.g., `element.className.split(" ").includes("active")`, but this is more error-prone and verbose than simply calling `element.classList.contains("active")`.

**Q12. What does `navigator.userAgent` tell you, and why is relying on it discouraged?**
It returns a string identifying the browser, rendering engine, and OS. It's discouraged for feature detection because user agent strings are inconsistent, frequently spoofed, and don't reliably indicate what APIs a browser actually supports — feature detection (checking if an API exists) is preferred instead.
