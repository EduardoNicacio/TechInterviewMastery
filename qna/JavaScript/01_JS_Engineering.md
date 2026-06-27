# JavaScript – Interview Questions & Answers

**Question**: How does the JavaScript event loop work?

**Answer**: The event loop continuously checks the call stack and task queues. When the call stack is empty, it first processes all microtasks (Promise callbacks, queueMicrotask), then picks one macrotask from the task queue (setTimeout, setInterval, I/O). This single-threaded model enables non-blocking async execution.

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2
```

---

**Question**: What is the difference between `setTimeout(fn, 0)` and `Promise.resolve().then()`?

**Answer**: `Promise.resolve().then()` queues a microtask that runs before the next macrotask, while `setTimeout(fn, 0)` queues a macrotask. Microtasks execute immediately after the current synchronous code and before any macrotask, making Promise callbacks strictly higher priority.

---

**Question**: What are closures and what are their common use cases?

**Answer**: A closure is a function that retains access to its outer lexical scope even after the outer function has returned. Common use cases include data encapsulation (private variables), function factories, and maintaining state in callbacks or event handlers.

```javascript
function createCounter() {
  let count = 0;
  return () => ++count;
}
const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

---

**Question**: How is the `this` keyword determined in JavaScript?

**Answer**: `this` is determined by the call site at runtime based on invocation rules: default binding (global/window, or undefined in strict mode), implicit binding (object method call), explicit binding (call/apply/bind), and `new` binding. Arrow functions inherit `this` lexically from their enclosing scope.

---

**Question**: What is the difference between `call()`, `apply()`, and `bind()`?

**Answer**: `fn.call(thisArg, arg1, arg2)` invokes `fn` with a given `this` context and comma-separated arguments. `fn.apply(thisArg, [args])` does the same with an argument array. `fn.bind(thisArg)` returns a new function with a permanently bound `this` without immediate invocation.

```javascript
function greet(greeting) { return `${greeting}, ${this.name}`; }
const user = { name: 'Alice' };
console.log(greet.call(user, 'Hello'));    // Hello, Alice
console.log(greet.apply(user, ['Hi']));    // Hi, Alice
const bound = greet.bind(user);
console.log(bound('Hey'));                 // Hey, Alice
```

---

**Question**: How does prototypal inheritance differ from classical inheritance?

**Answer**: In prototypal inheritance, objects inherit directly from other objects via the prototype chain, with no classes involved. Classical inheritance uses class blueprints where instances inherit from a class hierarchy. JavaScript's prototypal model is more flexible, allowing dynamic modification of prototypes at runtime.

```javascript
const animal = { speak() { return '...'; } };
const dog = Object.create(animal);
dog.bark = () => 'Woof!';
console.log(dog.speak()); // inherits from animal
```

---

**Question**: How does the `class` syntax work under the hood?

**Answer**: ES6 `class` is syntactic sugar over JavaScript's prototypal inheritance. Methods defined in a class are added to the constructor's prototype, and the constructor itself is a regular function. Classes enforce `new` invocation and support `extends` for prototype chain linking.

```javascript
class Rectangle {
  constructor(w, h) { this.w = w; this.h = h; }
  area() { return this.w * this.h; }
}
// Equivalent:
function Rectangle(w, h) { this.w = w; this.h = h; }
Rectangle.prototype.area = function() { return this.w * this.h; };
```

---

**Question**: What are the differences between `var`, `let`, and `const` regarding hoisting?

**Answer**: All three are hoisted, but `var` is initialized to `undefined` while `let` and `const` are in a temporal dead zone (TDZ) until the declaration is reached. `var` is function-scoped, while `let` and `const` are block-scoped. `const` additionally prevents reassignment of the binding.

```javascript
console.log(a); // undefined (var hoisted)
console.log(b); // ReferenceError: TDZ
var a = 1;
let b = 2;
```

---

**Question**: What are the differences between arrow functions and regular functions?

**Answer**: Arrow functions do not have their own `this`, `arguments`, `super`, or `new.target`, and cannot be used as constructors. They lexically bind `this` from the enclosing scope. Regular functions have dynamic `this` and can be used with `new`.

---

**Question**: What are spread and rest operators and how do they differ?

**Answer**: The spread operator (`...`) expands an iterable into individual elements, while the rest parameter collects remaining arguments into an array. Spread is used in function calls, arrays, and object literals; rest is used in function parameters and destructuring patterns.

```javascript
// Spread
const arr = [1, 2, 3];
console.log(Math.max(...arr)); // 3
// Rest
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
console.log(sum(1, 2, 3)); // 6
```

---

**Question**: How does destructuring work for objects and arrays?

**Answer**: Destructuring unpacks values from arrays or properties from objects into distinct variables using a pattern-matching syntax. It supports default values, nested destructuring, and renaming for objects.

```javascript
const [a, b] = [1, 2];
const { name: userName, age = 25 } = { name: 'Bob' };
console.log(userName, age); // Bob 25
```

---

**Question**: What features do template literals provide?

**Answer**: Template literals support string interpolation with `${expression}`, multi-line strings without concatenation, and tagged templates for custom string processing. They are enclosed by backticks and evaluated at runtime.

```javascript
const name = 'World';
console.log(`Hello, ${name}!`); // Hello, World!
```

---

**Question**: What is the difference between ESM and CommonJS modules?

**Answer**: ESM uses `import`/`export` syntax, is statically analyzable enabling tree shaking, has strict mode by default, supports top-level await, and is natively supported in modern browsers and Node.js. CommonJS uses `require()`/`module.exports`, is dynamic and synchronous, and is the Node.js legacy standard.

---

**Question**: What is the difference between `import` and `require()`?

**Answer**: `import` is static (hoisted, must be at top level) and works asynchronously in browsers, enabling tree shaking and dead code elimination. `require()` is dynamic (can be called conditionally inside functions) and synchronous, loading modules at runtime.

---

**Question**: What are the states of a Promise and how does chaining work?

**Answer**: A Promise has three states: pending, fulfilled, and rejected. Chaining uses `.then()` which returns a new Promise, allowing sequential async operations. Errors propagate through the chain and can be caught with `.catch()` at the end.

```javascript
fetch('/api/user')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

**Question**: How do you handle errors with async/await?

**Answer**: Use try/catch blocks around await expressions, or attach `.catch()` to the returned promise. For centralized handling, catch errors at the top-level caller or use a wrapper function that converts rejected promises to caught errors.

```javascript
async function getData() {
  try {
    const res = await fetch('/api');
    return await res.json();
  } catch (err) {
    console.error('Fetch failed:', err);
  }
}
```

---

**Question**: What are the Promise combinator methods?

**Answer**: `Promise.all` resolves when all promises resolve or rejects immediately on any rejection. `Promise.allSettled` waits for all to settle and returns results regardless. `Promise.race` settles with the first settled promise. `Promise.any` resolves with the first fulfilled promise or rejects if all reject.

```javascript
const p1 = Promise.resolve(1);
const p2 = Promise.reject('err');
Promise.allSettled([p1, p2]).then(results =>
  console.log(results.map(r => r.status)) // ['fulfilled', 'rejected']
);
```

---

**Question**: How do generators and iterators work in JavaScript?

**Answer**: Generators are functions (`function*`) that can pause execution with `yield` and resume later, maintaining their state. They return an iterator object with `next()` that produces `{ value, done }` pairs. Iterators implement the `[Symbol.iterator]` protocol for custom iteration.

```javascript
function* idGenerator() {
  let id = 0;
  while (true) yield ++id;
}
const gen = idGenerator();
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
```

---

**Question**: What are Symbols and their use cases?

**Answer**: Symbols are unique, immutable primitive values used as property keys to avoid name collisions. Common use cases include defining well-known protocols (`Symbol.iterator`), creating private-ish object properties, and implementing metaprogramming patterns.

```javascript
const key = Symbol('id');
const obj = { [key]: 123 };
console.log(obj[key]); // 123
console.log(Object.keys(obj)); // [] - Symbol keys are hidden
```

---

**Question**: What are the differences between Map, Set, WeakMap, and WeakSet?

**Answer**: Map holds key-value pairs with any type (including objects) as keys, preserving insertion order. Set stores unique values. WeakMap and WeakSet hold weak references to object keys, meaning they do not prevent garbage collection, and have no size property or iteration methods.

```javascript
const map = new Map();
map.set({}, 'value');
const wm = new WeakMap();
const obj = {};
wm.set(obj, 'value'); // obj can be GC'd after the reference is removed
```

---

**Question**: What are the key array methods and their use cases?

**Answer**: `map` transforms each element, `filter` selects matching elements, `reduce` accumulates to a single value, `find` returns the first match, `some` checks if any element passes, `every` checks if all pass. These enable a declarative, functional style without mutations.

```javascript
const nums = [1, 2, 3, 4, 5];
const doubled = nums.map(n => n * 2);            // [2, 4, 6, 8, 10]
const evens = nums.filter(n => n % 2 === 0);     // [2, 4]
const sum = nums.reduce((a, b) => a + b, 0);     // 15
const firstEven = nums.find(n => n % 2 === 0);   // 2
```

---

**Question**: How do you deep clone an object in JavaScript?

**Answer**: `structuredClone()` creates a deep copy handling circular references, Dates, Maps, Sets, and TypedArrays. The spread operator (`...`) and `Object.assign()` only do shallow copies. `JSON.parse(JSON.stringify(obj))` is a common hack but fails with functions, undefined, Symbols, and circular references.

```javascript
const obj = { a: 1, b: { c: 2 } };
const deep = structuredClone(obj);
const shallow = { ...obj };
shallow.b.c = 99;
console.log(obj.b.c); // 99 (shallow copy shares nested refs, so obj.b.c was mutated)
```

---

**Question**: What is currying and how does it differ from partial application?

**Answer**: Currying transforms a function taking multiple arguments into a sequence of nested functions each taking a single argument. Partial application pre-fills some arguments of a function, returning a function that takes the remaining arguments. Currying always produces unary functions; partial application does not.

```javascript
// Currying
const add = a => b => a + b;
add(2)(3); // 5
// Partial application
const bind = (fn, ...args) => (...rest) => fn(...args, ...rest);
```

---

**Question**: What is memoization and how does it improve performance?

**Answer**: Memoization caches the results of expensive function calls based on their arguments, avoiding redundant computation. It is commonly implemented with a Map or object cache keyed by arguments, and is useful for recursive algorithms like Fibonacci or dynamic programming.

```javascript
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (!cache.has(key)) cache.set(key, fn(...args));
    return cache.get(key);
  };
}
```

---

**Question**: What is the difference between debouncing and throttling?

**Answer**: Debouncing delays execution until a specified quiet period after the last event, useful for search inputs and auto-save. Throttling ensures execution at most once per interval, useful for scroll/resize handlers. Debouncing coalesces rapid bursts into a single call; throttling spreads calls evenly over time.

```javascript
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

function throttle(fn, interval) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= interval) { last = now; fn(...args); }
  };
}
```

---

**Question**: What is the difference between `null`, `undefined`, and undeclared?

**Answer**: `null` is an intentional absence of an object value (explicitly assigned). `undefined` means a variable has been declared but not assigned a value. Undeclared refers to a variable that has not been declared in any accessible scope, causing a ReferenceError.

---

**Question**: How does type coercion work in JavaScript?

**Answer**: JavaScript automatically converts types in operations like `==` comparisons, arithmetic, and string concatenation. Falsy values include `false`, `0`, `''`, `null`, `undefined`, `NaN`. All other values are truthy. Explicit coercion uses `Boolean()`, `Number()`, `String()`.

```javascript
console.log(5 + '5'); // '55' (number coerced to string)
console.log('5' - 3); // 2 (string coerced to number)
console.log(Boolean('')); // false
```

---

**Question**: What is the difference between `==` and `===`?

**Answer**: `==` performs type coercion before comparison, while `===` (strict equality) checks both value and type without coercion. Always prefer `===` except in rare cases where you intentionally leverage coercion (e.g., checking `null == undefined` with `==`).

```javascript
console.log(1 == '1');  // true (coercion)
console.log(1 === '1'); // false (no coercion)
console.log(null == undefined); // true (special case)
```

---

**Question**: What is the difference between `typeof` and `instanceof`?

**Answer**: `typeof` returns a string indicating the primitive type of a value (`'string'`, `'number'`, `'object'`, etc.). `instanceof` checks whether an object's prototype chain includes a constructor's prototype, making it suitable for checking custom class instances.

```javascript
typeof 'hello';      // 'string'
typeof null;         // 'object' (historical bug)
[] instanceof Array; // true
```

---

**Question**: How do you handle errors with try/catch and create custom errors?

**Answer**: Use try/catch/finally to handle runtime exceptions. Custom errors extend `Error` to add context like status codes or error codes. Always catch specific error types when possible and avoid swallowing errors silently.

```javascript
class HttpError extends Error {
  constructor(status, message) {
    super(message);
    this.status = status;
  }
}
try {
  throw new HttpError(404, 'Not found');
} catch (err) {
  if (err instanceof HttpError) console.error(err.status);
}
```

---

**Question**: How does event delegation work?

**Answer**: Event delegation leverages event bubbling by attaching a single listener to a parent element to handle events from multiple children. The event target is determined via `event.target`, eliminating the need for individual listeners and improving performance for dynamic lists.

```javascript
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log('Item clicked:', e.target.textContent);
  }
});
```

---

**Question**: What is the difference between `requestAnimationFrame` and `requestIdleCallback`?

**Answer**: `requestAnimationFrame` schedules a callback before the next paint, ideal for animations and visual updates. `requestIdleCallback` schedules a callback when the main thread is idle, suitable for non-critical work like analytics or deferred processing.

---

**Question**: What are Web Workers and when should you use them?

**Answer**: Web Workers run scripts in background threads separate from the main UI thread, enabling parallel execution for CPU-intensive tasks. They communicate via `postMessage` and have no access to the DOM, making them suitable for data processing, encryption, or image manipulation.

---

**Question**: What are Service Workers and what caching strategies are common?

**Answer**: Service Workers are proxy scripts that intercept network requests, enabling offline support, push notifications, and background sync. Common caching strategies include Cache First (static assets), Network First (dynamic content), Stale-While-Revalidate (frequent updates), and Network Only (sensitive data).

---

**Question**: What causes memory leaks in JavaScript and how do you prevent them?

**Answer**: Common causes include detached DOM references (elements removed from DOM but still referenced), uncleaned timers/intervals, closures holding large objects, global variables, and forgotten event listeners. Prevent leaks by cleaning up references in lifecycle hooks, using WeakRef/WeakMap, and profiling with Chrome DevTools.

---

**Question**: What are the phases of the Node.js event loop?

**Answer**: The Node.js event loop has six phases: timers (setTimeout/setInterval), pending callbacks (I/O callbacks), idle/prepare (internal), poll (I/O events), check (setImmediate), and close callbacks (socket onclose). Each phase runs its queue before moving to the next, with microtasks processed between phases.

---

**Question**: How do Node.js streams work?

**Answer**: Streams are event-based interfaces for handling data flow piece by piece, avoiding large memory consumption. Readable streams emit `data` and `end` events. Writable streams accept `write()` calls. Transform streams (e.g., zlib Gzip) are both readable and writable, modifying data in transit.

```javascript
const { Readable, Transform } = require('stream');
const upper = new Transform({
  transform(chunk, enc, cb) { cb(null, chunk.toString().toUpperCase()); }
});
process.stdin.pipe(upper).pipe(process.stdout);
```

---

**Question**: What are Buffers and TypedArrays?

**Answer**: Buffers (Node.js) and TypedArrays (Uint8Array, Float64Array, etc.) provide raw binary data access with fixed-length memory allocation. Buffers extend Uint8Array with Node-specific methods like `toString()`. TypedArrays are ECMAScript standard and are used for WebGL, WebAudio, and binary protocol parsing.

---

**Question**: How does REST differ from GraphQL?

**Answer**: REST uses fixed endpoints where each request returns a predefined data structure. GraphQL uses a single endpoint where clients specify exactly which fields they need, reducing over-fetching and under-fetching. GraphQL supports real-time subscriptions and has a strong type system via its schema.

---

**Question**: What is CORS and how do preflight requests work?

**Answer**: Cross-Origin Resource Sharing (CORS) is a browser security mechanism that controls cross-origin HTTP requests via headers. Preflight requests (OPTIONS) are sent before non-simple requests (custom headers, non-standard methods, or non-urlencoded content-types) to check server permissions.

---

**Question**: What is the Same-Origin Policy?

**Answer**: The Same-Origin Policy prevents scripts on one origin (protocol + domain + port) from accessing resources from another origin. It is a fundamental browser security measure that blocks cross-origin reads, though cross-origin writes (forms, links) and embedding (images, scripts) are typically allowed.

---

**Question**: What is Content Security Policy (CSP)?

**Answer**: CSP is an HTTP header (`Content-Security-Policy`) that restricts which resources (scripts, styles, images, fonts) a page can load and execute. It mitigates XSS attacks by whitelisting trusted sources and disabling inline scripts or eval unless explicitly allowed.

---

**Question**: What are the differences between WebSocket and Server-Sent Events (SSE)?

**Answer**: WebSocket provides full-duplex communication over a persistent TCP connection, suitable for real-time bidirectional apps (chat, gaming). SSE is unidirectional from server to client over HTTP, automatically reconnects, and is simpler for push notifications or live feeds. SSE only works with text data; WebSocket supports binary.

---

**Question**: What are tree shaking and code splitting?

**Answer**: Tree shaking is dead-code elimination via static analysis of ESM imports, removing unused exports during bundling. Code splitting breaks the bundle into smaller chunks loaded on demand, reducing initial load time. Both are typically configured in bundlers like Webpack, Rollup, or Vite.

---

**Question**: How do lazy loading and dynamic imports work?

**Answer**: Dynamic imports (`import()`) load modules on demand as Promises, enabling lazy loading. They split the bundle at the import point, and the module code is fetched and executed only when the import expression evaluates. This is commonly used for route-based code splitting in SPAs.

```javascript
button.addEventListener('click', async () => {
  const module = await import('./heavyModule.js');
  module.run();
});
```

---

**Question**: What are Core Web Vitals (LCP, INP, CLS)?

**Answer**: Largest Contentful Paint (LCP) measures loading performance (visible main content). Interaction to Next Paint (INP) measures responsiveness (latency of all interactions throughout the page lifecycle, reporting the worst-case). Cumulative Layout Shift (CLS) measures visual stability (unexpected layout shifts). These metrics are Google's ranking signals for user experience.

---

**Question**: How do Webpack, Rollup, and Vite differ in bundling?

**Answer**: Webpack is feature-rich with extensive plugin ecosystem but slower for development. Rollup optimizes for ESM with superior tree shaking, commonly used for libraries. Vite uses native ESM in development for instant hot module replacement and Rollup for production builds, offering the fastest dev experience.

---

**Question**: How do you implement the Singleton pattern in JavaScript?

**Answer**: The Singleton pattern ensures only one instance of a class exists. In JavaScript, it can be implemented using a module-level instance variable, a static getInstance method, or simply exporting a single object literal from a module (module singletons).

```javascript
class Singleton {
  static #instance;
  constructor() {
    if (Singleton.#instance) return Singleton.#instance;
    Singleton.#instance = this;
  }
}
```

---

**Question**: How do you implement the Observer pattern in JavaScript?

**Answer**: The Observer pattern maintains a list of subscribers and notifies them when state changes. It can be implemented with a simple class managing subscriber callbacks, or using built-in EventEmitter in Node.js. This pattern is the foundation for reactive programming and event-driven architectures.

```javascript
class Observable {
  constructor() { this._subscribers = new Set(); }
  subscribe(fn) { this._subscribers.add(fn); }
  unsubscribe(fn) { this._subscribers.delete(fn); }
  notify(data) { this._subscribers.forEach(fn => fn(data)); }
}
```

---

**Question**: What are the Module pattern and IIFE pattern?

**Answer**: The Module pattern encapsulates private state and exposes a public API, typically using an IIFE (Immediately Invoked Function Expression) that returns an object. This creates a closure for private variables, preventing global scope pollution. Modern ESM modules supersede this pattern with native encapsulation.

```javascript
const CounterModule = (() => {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count
  };
})();
console.log(CounterModule.increment()); // 1
```