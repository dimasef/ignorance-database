Closure is when a function remembers variables from an outer scope even if that outer function has already terminated.

## 1. Counter (private state)

```js
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.getCount(); // 2
// count is not accessible from outside — it's "closed over"
```

Used in: modules with private state, React hooks work this way under the hood.

## 2. Function factory (partial application)

```js
function createLogger(prefix) {
  return function (message) {
    console.log(`[${prefix}] ${message}`);
  };
}

const errorLogger = createLogger("ERROR");
const infoLogger = createLogger("INFO");

errorLogger("Something broke"); // [ERROR] Something broke
```

Used in: utilities, middleware, configs — when you need to "remember" part of the arguments.

## 3. Event handlers

```js
function setupButton(buttonId, message) {
  const btn = document.getElementById(buttonId);
  btn.addEventListener("click", () => {
    console.log(message); // message is closed over
  });
}

setupButton("saveBtn", "Saved!");
setupButton("deleteBtn", "Deleted!");
```

Used in: any UI code — each handler closes over its own data.

## 4. Debounce

```js
function debounce(fn, delay) {
  let timerId;
  return (...args) => {
    clearTimeout(timerId);
    timerId = setTimeout(() => fn(...args), delay);
  };
}

const search = debounce((query) => fetch(`/api?q=${query}`), 300);
```

Used in: search inputs, resize/scroll handlers — one of the most common utilities.

## 5. Caching (memoization)

```js
function memoize(fn) {
  const cache = {};
  return (arg) => {
    if (arg in cache) return cache[arg];
    return (cache[arg] = fn(arg));
  };
}

const heavyCalc = memoize((n) => {
  console.log("computing...");
  return n * n;
});

heavyCalc(5); // "computing..." → 25
heavyCalc(5); // 25 (from cache, no log)
```

Used in: computation optimization, `React.useMemo` / `useCallback` are built on this principle.
