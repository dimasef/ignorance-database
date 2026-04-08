Lexical scope means that a variable's accessibility is determined by where it is written in the source code, not by how or where the function is called at runtime.

An inner function can access variables from its outer function simply because it is defined inside it.

## 1. Nested functions

```js
function outer() {
  const name = "Alice";

  function inner() {
    console.log(name); // "Alice" — visible because inner is defined inside outer
  }

  inner();
}

outer();
```

The variable `name` is accessible inside `inner` because `inner` is lexically (physically) nested within `outer`.

## 2. Scope chain

```js
const global = "I am global";

function first() {
  const a = "I am from first";

  function second() {
    const b = "I am from second";

    function third() {
      console.log(b);      // "I am from second"
      console.log(a);      // "I am from first"
      console.log(global); // "I am global"
    }

    third();
  }

  second();
}

first();
```

When a variable is not found in the current scope, the engine walks up the lexical scope chain — from inner to outer — until it finds it or reaches the global scope.

## 3. Lexical scope vs dynamic scope

```js
function logX() {
  console.log(x); // looks for x where logX is DEFINED, not where it is called
}

function caller() {
  const x = 42;
  logX(); // ReferenceError — x is not in logX's lexical scope
}

const x = 10;
caller(); // 10 — logX sees the global x, not caller's x
```

JavaScript uses lexical (static) scope: what matters is where the function was written, not where it was invoked. Most languages work this way. Dynamic scope (used in some Lisps, Bash) would resolve `x` based on the call stack instead.

## 4. Block scope with let/const

```js
function example() {
  if (true) {
    let blockScoped = "only here";
    const alsoBlock = "same";
    var functionScoped = "whole function";
  }

  console.log(functionScoped); // "whole function"
  console.log(blockScoped);    // ReferenceError
}
```

`let` and `const` are scoped to the nearest `{}` block. `var` is scoped to the nearest function. Both follow lexical scoping — the boundary is determined by source code structure.

## Connection to closures

Lexical scope is the foundation of closures. A closure is a function that retains access to its lexical environment. Without lexical scoping, closures would not work — the function would not know which variables to "remember."
