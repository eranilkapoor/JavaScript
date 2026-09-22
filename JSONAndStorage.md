**JSON (JavaScript Object Notation)** is a lightweight, text-based data format used to exchange data between a client and a server, and to store structured data in the browser. It's derived from JavaScript object syntax but is language-independent — nearly every programming language has a JSON library. This file covers JSON itself, along with the browser storage mechanisms (`localStorage`, `sessionStorage`, cookies) that commonly store it.

---

### **What Is JSON?**
JSON represents data as text using a small set of structures: objects (`{}`), arrays (`[]`), strings, numbers, booleans, and `null`. It's the standard format for APIs, config files, and browser storage.

```json
{
  "name": "Alice",
  "age": 30,
  "isActive": true,
  "roles": ["admin", "editor"],
  "address": {
    "city": "Boston"
  },
  "manager": null
}
```

#### **Valid JSON Syntax Rules**
JSON looks like a JavaScript object literal, but it's stricter:

1. **Keys must be double-quoted strings** — `{name: "Alice"}` is invalid JavaScript-object-as-JSON; it must be `{"name": "Alice"}`.
2. **Strings must use double quotes**, not single quotes.
3. **No trailing commas** — `["a", "b",]` is invalid JSON.
4. **No comments** are allowed anywhere in JSON.
5. **No functions, `undefined`, or `Symbol`** — JSON only supports strings, numbers, booleans, `null`, objects, and arrays.
6. **No `Date` objects, `Map`, or `Set`** directly — these must be converted to a supported type (usually a string) before being represented in JSON.

```javascript
// Invalid JSON (this is valid JS, but NOT valid JSON text):
// { name: 'Alice', age: 30, } <- unquoted key, single quotes, trailing comma
```

---

### **`JSON.stringify()`**
Converts a JavaScript value into a JSON string.

```javascript
const user = { name: "Alice", age: 30, isActive: true };
const json = JSON.stringify(user);

console.log(json);
// Output: {"name":"Alice","age":30,"isActive":true}
console.log(typeof json);
// Output: string
```

#### **The `replacer` Argument**
The second argument can filter or transform properties — either an array of allowed keys, or a function called for every key-value pair.

```javascript
const user = { name: "Alice", age: 30, password: "secret123" };

// Array form: only include listed keys
console.log(JSON.stringify(user, ["name", "age"]));
// Output: {"name":"Alice","age":30}

// Function form: transform or omit values
const safeJson = JSON.stringify(user, (key, value) => {
  if (key === "password") return undefined; // omit this key
  return value;
});
console.log(safeJson);
// Output: {"name":"Alice","age":30}
```

#### **The `indent` (Space) Argument**
The third argument pretty-prints the output with the given number of spaces (or a string) per indentation level — useful for logging or writing readable config files.

```javascript
console.log(JSON.stringify(user, null, 2));
// Output:
// {
//   "name": "Alice",
//   "age": 30,
//   "password": "secret123"
// }
```

---

### **`JSON.parse()`**
Converts a JSON string back into a JavaScript value.

```javascript
const json = '{"name":"Alice","age":30}';
const user = JSON.parse(json);

console.log(user.name, user.age);
// Output: Alice 30
console.log(typeof user);
// Output: object
```

#### **The `reviver` Argument**
A function that transforms each key-value pair as it's parsed — useful for reconstructing types JSON doesn't natively support, like `Date`.

```javascript
const json = '{"name":"Alice","createdAt":"2024-06-15T10:00:00.000Z"}';

const user = JSON.parse(json, (key, value) => {
  if (key === "createdAt") return new Date(value);
  return value;
});

console.log(user.createdAt instanceof Date);
// Output: true
```

---

### **Common Gotchas**

1. **`undefined`, functions, and `Symbol` are dropped** (in objects) or converted to `null` (in arrays) during `stringify()`:
```javascript
const data = { a: 1, b: undefined, c: () => {}, d: Symbol("x") };
console.log(JSON.stringify(data));
// Output: {"a":1}

console.log(JSON.stringify([1, undefined, () => {}]));
// Output: [1,null,null]
```

2. **Circular references throw an error**:
```javascript
const obj = { name: "Alice" };
obj.self = obj; // circular reference

try {
  JSON.stringify(obj);
} catch (error) {
  console.log(error.message);
  // Output: Converting circular structure to JSON...
}
```

3. **`Date` objects are converted to ISO strings**, not preserved as dates — they must be manually revived back into `Date` objects after parsing (see `reviver` above).
```javascript
console.log(JSON.stringify({ when: new Date(2024, 0, 1) }));
// Output: {"when":"2024-01-01T00:00:00.000Z"} (exact time depends on timezone)
```

4. **`NaN` and `Infinity` become `null`** since JSON has no representation for them:
```javascript
console.log(JSON.stringify({ value: NaN }));
// Output: {"value":null}
```

---

### **`localStorage` vs `sessionStorage`**
Both are part of the **Web Storage API**, storing key-value pairs (as strings) directly in the browser, scoped to the page's origin (protocol + domain + port).

#### **The API (identical for both)**
```javascript
localStorage.setItem("theme", "dark");
console.log(localStorage.getItem("theme"));
// Output: dark

localStorage.removeItem("theme");
console.log(localStorage.getItem("theme"));
// Output: null

localStorage.setItem("a", "1");
localStorage.setItem("b", "2");
localStorage.clear(); // removes everything
```

`sessionStorage` uses the exact same methods:
```javascript
sessionStorage.setItem("draftText", "Hello world");
console.log(sessionStorage.getItem("draftText"));
// Output: Hello world
```

#### **Storing Objects (Must Stringify/Parse)**
Web Storage only stores strings — objects must be serialized manually.
```javascript
const settings = { theme: "dark", fontSize: 14 };

localStorage.setItem("settings", JSON.stringify(settings));

const saved = JSON.parse(localStorage.getItem("settings"));
console.log(saved.theme, saved.fontSize);
// Output: dark 14
```

#### **Persistence Difference**
- **`localStorage`**: Data persists indefinitely — across tabs, browser restarts, and even system reboots — until explicitly cleared by code or the user.
- **`sessionStorage`**: Data persists only for the lifetime of that specific browser tab — it's cleared when the tab is closed (but survives page reloads within the same tab).

---

### **Cookies**
Cookies are small pieces of data (historically capped around **4KB**) that are automatically attached to every HTTP request to their matching domain — meaning, unlike Web Storage, **the server can read and set them directly**.

```javascript
// Setting a cookie
document.cookie = "username=Alice; max-age=3600; path=/";

// Reading cookies (returns ALL cookies as one semicolon-separated string)
console.log(document.cookie);
// Output: username=Alice; theme=dark  (example — all cookies concatenated)
```

Working with `document.cookie` directly is clunky (parsing the string, handling expiry, encoding values), so most real projects use a small helper library rather than manipulating it by hand.

#### **When Cookies Are Still Needed**
- **The server needs to read the value** on every request (e.g., session/auth tokens) — `localStorage`/`sessionStorage` are never sent to the server automatically.
- **Precise expiry control** is needed (`max-age`, `expires`) beyond "until cleared" or "until tab closes."
- **Cross-request behavior** matters, such as CSRF protection tokens or server-rendered personalization.

---

### **Comparison Table**
| | `localStorage` | `sessionStorage` | Cookies |
|---|---|---|---|
| **Persistence** | Until explicitly cleared | Until the tab is closed | Configurable (`max-age`/`expires`), can be persistent or session-only |
| **Size limit** | ~5-10MB (browser-dependent) | ~5-10MB (browser-dependent) | ~4KB |
| **Sent to server automatically** | No | No | Yes, on every matching HTTP request |
| **Accessible from JS** | Yes (`localStorage` API) | Yes (`sessionStorage` API) | Yes (`document.cookie`), unless `HttpOnly` is set (server-only cookies) |
| **Scope** | Per origin | Per origin + per tab | Per domain/path, configurable |
| **Typical use** | User preferences, cached data, drafts | Temporary per-tab state (form progress, wizard steps) | Auth/session tokens, server-read data |

---

### **Best Practices**
- Always wrap `JSON.parse()` calls on external or storage data in `try/catch` — malformed or corrupted data will throw.
- Use `localStorage`/`sessionStorage` for client-only data (UI preferences, drafts, cached API responses); use cookies when the server must read the value.
- Never store sensitive data (passwords, raw tokens) in `localStorage` — it's readable by any JavaScript running on the page, making it vulnerable to XSS attacks. Prefer `HttpOnly` cookies for sensitive session tokens.
- Namespace your storage keys (e.g., `"myApp:theme"`) to avoid collisions with other scripts or future features.
- Remember Web Storage is synchronous — avoid storing very large objects, as it can block the main thread.
- Check `typeof window !== "undefined"` (or similar) before accessing storage APIs in code that might run in non-browser environments (like server-side rendering).

---

### **Interview Questions**

**Q1. What is JSON, and why is it so widely used?**
JSON (JavaScript Object Notation) is a lightweight, text-based data format for representing structured data using objects, arrays, strings, numbers, booleans, and `null`. It's widely used because it's human-readable, maps closely to native data structures in most languages, and has broad, standardized library support for APIs and configuration.

**Q2. What's the difference between JSON and a JavaScript object literal?**
JSON is a strict text format (keys must be double-quoted strings, no trailing commas, no comments, no functions) while a JS object literal is more permissive JavaScript syntax (unquoted keys, single quotes, trailing commas, and values like functions are all allowed). Every valid JSON document is valid JS, but not every JS object literal is valid JSON.

**Q3. What happens when you `JSON.stringify()` a value containing `undefined` or a function?**
In an object, keys whose value is `undefined` or a function are simply omitted from the output. In an array, they're converted to `null` instead of being dropped, since arrays must preserve their length/order.

**Q4. What happens if you try to `JSON.stringify()` an object with a circular reference?**
It throws a `TypeError` ("Converting circular structure to JSON") because `stringify()` can't represent an object that references itself, directly or indirectly, without infinite recursion.

**Q5. What does the `replacer` parameter of `JSON.stringify()` do?**
It filters or transforms the data being serialized. As an array, it's a whitelist of keys to include. As a function, it's called for every key-value pair and can return a modified value or `undefined` to omit that key entirely — commonly used to strip sensitive fields like passwords.

**Q6. How do you make `JSON.parse()` reconstruct a `Date` object instead of leaving it as a string?**
Pass a `reviver` function as the second argument that checks the key name and converts the string back into a `Date`:
```javascript
JSON.parse(json, (key, value) => key === "createdAt" ? new Date(value) : value);
```

**Q7. Why can't you store an object directly in `localStorage`?**
`localStorage` only stores strings. Attempting to store an object directly stores its string coercion (`"[object Object]"`), losing the data. You must `JSON.stringify()` it before storing, and `JSON.parse()` it after retrieving.

**Q8. What's the key difference between `localStorage` and `sessionStorage`?**
Both share the same API, but `localStorage` persists indefinitely until explicitly cleared, while `sessionStorage` is cleared automatically when the browser tab is closed (though it survives page reloads within that tab).

**Q9. Why would you use a cookie instead of `localStorage`?**
Because cookies are automatically included in every HTTP request to the matching domain, so the server can read them directly — essential for things like session/auth tokens. `localStorage` is never transmitted to the server automatically.

**Q10. What's a security concern with storing auth tokens in `localStorage`?**
Any JavaScript running on the page — including malicious code injected via an XSS vulnerability — can read `localStorage` freely. Sensitive tokens are safer in an `HttpOnly` cookie, which JavaScript cannot access at all.

**Q11. Roughly how much data can you store in `localStorage` versus a cookie?**
`localStorage` typically allows a few megabytes (commonly 5-10MB depending on the browser), while a single cookie is limited to roughly 4KB, and all cookies for a domain are sent with every request, adding overhead to network calls.

**Q12. Why does `JSON.stringify()` turn `NaN` and `Infinity` into `null`?**
Because JSON has no native representation for non-finite numbers — its number type only supports standard finite numeric literals — so `stringify()` falls back to `null` for `NaN`, `Infinity`, and `-Infinity` rather than producing invalid JSON.
