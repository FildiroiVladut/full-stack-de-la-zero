# Întrebări JavaScript și TypeScript

32 de întrebări care apar des la interviurile de junior pe web. Întrebarea e în engleză, cum o vei auzi. Răspunsul-model e tot în engleză și are 3–6 propoziții, cât spui în 30–60 de secunde. Notele în română explică de ce se pune întrebarea sau unde e capcana.

Citește întrebarea, răspunde cu voce tare, abia apoi deschide răspunsul.

## Fundamente

Detalii: [04-javascript/01-recap-zilele-1-7.md](../04-javascript/01-recap-zilele-1-7.md), [04-javascript/06-js-in-cod-real.md](../04-javascript/06-js-in-cod-real.md)

<details>
<summary><b>1. What's the difference between <code>==</code> and <code>===</code>?</b></summary>

`===` compares value and type with no conversion. `==` converts the operands to a common type first, and the rules are hard to remember: `'1' == 1` is true, `0 == ''` is true, `null == undefined` is true. I always use `===`. The only place some codebases use `==` on purpose is `x == null`, which checks for both `null` and `undefined` at once.

_De ce se întreabă:_ e o întrebare de încălzire. Exemplele concrete arată că știi, nu doar că ai citit regula.
</details>

<details>
<summary><b>2. What's the difference between <code>var</code>, <code>let</code> and <code>const</code>?</b></summary>

`var` is function-scoped and hoisted with the value `undefined`, so you can read it before the line where it's declared. `let` and `const` are block-scoped. They're hoisted too, but they sit in the "temporal dead zone" until their declaration, so reading them earlier throws a `ReferenceError`. `const` prevents reassigning the variable, not mutating the object: `const items = []; items.push(1)` is fine. I default to `const`, use `let` when I need to reassign, and never use `var` in new code.

_Capcana:_ „`const` înseamnă imutabil” e greșit, și intervievatorii așteaptă exact confuzia asta.
</details>

<details>
<summary><b>3. How are primitives and objects passed to functions?</b></summary>

Primitives (string, number, boolean, null, undefined, bigint, symbol) are copied by value. Objects, arrays and functions are reference values: the variable holds a reference, so assigning one or passing it to a function shares the same object. Strictly speaking, JavaScript is always pass-by-value, but for objects the value being copied is the reference. That's why mutating an object parameter inside a function changes the caller's object, while reassigning the parameter doesn't.
</details>

<details>
<summary><b>4. What are truthy and falsy values? When do you use <code>??</code> instead of <code>||</code>?</b></summary>

The falsy values are `false`, `0`, `-0`, `0n`, `''`, `null`, `undefined` and `NaN`. Everything else is truthy, including `[]`, `{}` and `'0'`. `||` falls back on any falsy value, while `??` falls back only on `null` or `undefined`. So `quantity || 1` turns a real `0` into `1`, which is a bug, while `quantity ?? 1` keeps the `0`. For defaults, I use `??` unless I really want to replace empty strings and zeros too.
</details>

<details>
<summary><b>5. What is a closure? Give a practical example.</b></summary>

A closure is a function together with the variables from the scope where it was created. It keeps access to them even after the outer function has returned. Practical uses: private state, like a counter or an ID generator; function factories; event handlers and callbacks that remember values. In React, every event handler and effect is a closure over the props and state of the render it was created in, and that's where stale-closure bugs come from.

```js
function createIdGenerator(prefix) {
  let next = 1;
  return () => `${prefix}-${next++}`;
}
const nextInvoiceId = createIdGenerator('INV');
nextInvoiceId(); // 'INV-1'
```

_De ce se întreabă:_ e probabil cea mai frecventă întrebare de JS. Legătura cu React îți dă puncte în plus.
</details>

<details>
<summary><b>6. How does <code>this</code> work? How is it different in arrow functions?</b></summary>

`this` is decided by how a function is called, not where it's defined. Called as `obj.method()`, `this` is `obj`. Called as a plain function, it's `undefined` in strict mode, which is the default in modules and classes. With `new`, it's the new instance, and `call`, `apply` and `bind` set it explicitly. Arrow functions don't have their own `this`; they take it from the surrounding scope, which is why they're used for callbacks inside methods. The classic bug is passing `obj.method` as a callback: it's no longer called "on" the object, so `this` is lost.
</details>

<details>
<summary><b>7. Shallow copy vs deep copy: how do you make each?</b></summary>

Spread (`{ ...obj }`, `[...arr]`) and `Object.assign` make a shallow copy: the top level is new, but nested objects are still shared with the original. `structuredClone(obj)` makes a deep copy and handles nested objects, arrays, Dates, Maps and Sets. It throws on functions, and class instances come back as plain objects. The old `JSON.parse(JSON.stringify(x))` trick loses Dates, `undefined` and Maps. In React state updates, I usually spread at each level I'm changing.
</details>

<details>
<summary><b>8. When do you use <code>map</code>, <code>filter</code>, <code>reduce</code>, <code>forEach</code> and <code>find</code>?</b></summary>

`map` returns a new array of the same length with each element transformed. `filter` returns a new array with the elements that pass a test. `find` returns the first match or `undefined`. `reduce` folds an array into a single value: a sum, an object, a lookup map. `forEach` returns nothing and is only for side effects. If I'm building a new value, I use map, filter or reduce. For side effects, or when I need `break` or a sequential `await`, I use `for...of`. For grouping there's now `Object.groupBy`, which is part of ES2024.
</details>

<details>
<summary><b>9. What are prototypes, and what does <code>class</code> do under the hood?</b></summary>

Every object has an internal link to a prototype object. When a property isn't found on the object itself, JavaScript looks it up along the prototype chain. `class` is syntax on top of this: methods declared in the class body live on `ClassName.prototype` and are shared by all instances, while fields live on each instance. `extends` links the two prototypes. Private fields (`#total`) are real runtime privacy, unlike TypeScript's `private` keyword, which only exists at compile time.
</details>

<details>
<summary><b>10. What's the difference between <code>null</code> and <code>undefined</code>?</b></summary>

`undefined` means "not assigned": a missing property, a missing argument, a function with no return value. `null` is an intentional "no value", set by a programmer or an API. `typeof null === 'object'` is a historical bug. JSON has `null` but not `undefined`, so properties set to `undefined` disappear in `JSON.stringify`. In APIs and databases, I use `null` for "explicitly empty".
</details>

<details>
<summary><b>11. What does optional chaining do, and when can it hurt?</b></summary>

`a?.b?.c` stops and returns `undefined` if any link in the chain is `null` or `undefined`, instead of throwing. It also works for calls, `fn?.()`, and indexes, `arr?.[0]`. Combined with `??` it gives clean defaults: `user?.address?.city ?? 'Unknown'`. It hurts when it's overused: if a value should always exist, `?.` turns a loud crash at the source into a silent `undefined` that breaks something three files away.

_Capcana:_ codul generat de AI pune adesea `?.` peste tot. La review, întreabă-te dacă valoarea chiar poate lipsi.
</details>

<details>
<summary><b>12. Why is <code>0.1 + 0.2 !== 0.3</code>, and how do you handle money?</b></summary>

JavaScript numbers are IEEE 754 double-precision floats. `0.1` and `0.2` have no exact binary representation, so the sum is `0.30000000000000004`. For money, I store integers in the smallest unit (cents, or bani for lei), do the arithmetic on integers, and format only for display with `Intl.NumberFormat`. In the database, I use an integer or `numeric` column, never a float.

_De ce se întreabă:_ arată că ai lucrat cu date reale. În Invoicer ai făcut exact asta.
</details>

<details>
<summary><b>13. What does immutability mean, and why does it matter?</b></summary>

Immutability means you don't change an existing object or array; you create a new one with the change applied. It matters because React and many other tools detect changes by reference. If you push into the same array, the reference doesn't change and React may skip the re-render. It also makes code easier to reason about, because nothing changes behind your back. Newer array methods help: `toSorted`, `toReversed`, `toSpliced` and `with` return new arrays instead of mutating.
</details>

<details>
<summary><b>14. ES modules vs CommonJS, and named vs default exports?</b></summary>

ES modules use `import`/`export`. They're static, so tools can analyze them before running the code, which is what makes tree-shaking possible. They're strict by default and are the standard in browsers and modern Node (`"type": "module"` in `package.json`). CommonJS is Node's older system, with `require` and `module.exports`, resolved at runtime. You still see it in older packages and many existing backends. Named exports must be imported by their exact name, which helps refactoring and autocomplete. A default export can be imported under any name.
</details>

## Async

Detalii: [04-javascript/02-event-loop-promises.md](../04-javascript/02-event-loop-promises.md), [04-javascript/03-async-await-fetch.md](../04-javascript/03-async-await-fetch.md)

<details>
<summary><b>15. What is the event loop?</b></summary>

JavaScript runs on a single thread with one call stack. Slow work like timers, network requests and file I/O is handed to the environment, meaning the browser or libuv in Node, and when it finishes, its callback is queued. The event loop picks the next callback only when the call stack is empty. There are two main queues. Microtasks, such as promise callbacks, are drained completely after each task, before the next macrotask, such as `setTimeout`, I/O or UI events. The consequence: a long synchronous loop blocks everything. The UI freezes, or in Node the server stops answering every request.
</details>

**16. What does this print, and in what order?**

```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```

<details>
<summary>Answer</summary>

`A`, `D`, `C`, `B`. A and D are synchronous, so they run first. When the stack is empty, the microtask queue is drained, which prints C, the promise callback. Only then does the event loop take the next macrotask, the `setTimeout` callback, and print B. `setTimeout(fn, 0)` means "as soon as possible after the current work", not "immediately".

_De ce se întreabă:_ e varianta practică a întrebării 15. Explică pe rând fiecare coadă, nu da doar ordinea.
</details>

<details>
<summary><b>17. What is a Promise?</b></summary>

A Promise is an object that represents a value that will be available later. It starts as pending, then settles exactly once, either fulfilled with a value or rejected with a reason, and never changes after that. You consume it with `.then`, `.catch` and `.finally`, or with `await`. An `async` function always returns a Promise, even when you `return` a plain value.
</details>

<details>
<summary><b>18. <code>async/await</code> vs <code>.then()</code>: what's the difference?</b></summary>

`async/await` is syntax on top of promises. `await` pauses the async function until the promise settles, without blocking the thread. The code reads top to bottom, and errors are handled with `try/catch` instead of `.catch` chains. Under the hood it's the same mechanism. The common bugs are forgetting `await`, so you get a Promise instead of the value and the error becomes an unhandled rejection, and awaiting inside a loop when the calls could run in parallel.
</details>

<details>
<summary><b>19. <code>Promise.all</code>, <code>allSettled</code>, <code>race</code>, <code>any</code>: when do you use each?</b></summary>

`Promise.all` waits for all of them and rejects as soon as one rejects. I use it when I need every result. `allSettled` waits for all, never rejects, and gives me the status of each, which is good for batch jobs where partial failure is acceptable. `race` settles with the first promise to settle, fulfilled or rejected; the classic use is a timeout. `any` resolves with the first one that fulfills and rejects only if all of them reject, with an `AggregateError`.
</details>

**20. This loop is slow. Why, and how do you fix it?**

```js
const invoices = [];
for (const id of ids) {
  invoices.push(await fetchInvoice(id));
}
```

<details>
<summary>Answer</summary>

Each request waits for the previous one to finish, so ten 200 ms requests take two seconds. If the requests don't depend on each other, I start them all and wait together: `const invoices = await Promise.all(ids.map(fetchInvoice));`. That takes about as long as the slowest request. With hundreds of IDs, I'd limit concurrency, for example in batches of 5–10, so I don't hit the API's rate limits. If each request depends on the previous result, the sequential loop is the right choice.
</details>

<details>
<summary><b>21. Does <code>fetch</code> reject on a 404? How do you handle HTTP errors?</b></summary>

No. `fetch` rejects only on network failures: no connection, DNS failure, a request blocked by CORS. For a 404 or a 500, it resolves normally with `response.ok === false`. So I check `ok` before reading the body:

```js
const res = await fetch(url);
if (!res.ok) throw new Error(`Request failed: ${res.status}`);
const data = await res.json();
```

`res.json()` can also throw if the body isn't valid JSON.

_Capcana:_ e unul dintre cele mai frecvente bug-uri în codul de junior și în codul generat de AI.
</details>

<details>
<summary><b>22. How do you cancel a fetch, and why would you?</b></summary>

I create an `AbortController` and pass its `signal` to `fetch`. Calling `controller.abort()` rejects the fetch with an `AbortError`. Typical reasons: the user leaves the page; the user types a new search query, so you cancel the old request and a slow old response doesn't overwrite the new results; or you need a timeout, for which there's `AbortSignal.timeout(5000)`. In React, TanStack Query handles this for you. In a raw `useEffect`, you abort in the cleanup function.
</details>

<details>
<summary><b>23. Debounce vs throttle?</b></summary>

Debounce waits until events stop for X milliseconds and then runs once. It's the right tool for search-as-you-type: you send one request when the user pauses, not one per keystroke. Throttle runs at most once every X milliseconds while events keep coming, which suits scroll or mouse-move handlers. Both reduce wasted work and API calls.
</details>

## TypeScript

Detalii: [05-typescript/](../05-typescript/)

<details>
<summary><b>24. What is TypeScript, and what doesn't it do?</b></summary>

TypeScript is JavaScript with static types that are checked at compile time. The compiler, or a tool like Vite, strips the types and outputs plain JavaScript. Types don't exist at runtime, so TypeScript can't protect you from bad data coming from an API, a form or a database; for that you need runtime validation, with zod or class-validator. What it gives you: errors caught before running, autocomplete, safe refactors, and types that work as documentation.

_De ce se întreabă:_ partea cu „ce nu face” desparte un junior care a înțeles de unul care doar a folosit TypeScript.
</details>

<details>
<summary><b>25. <code>type</code> vs <code>interface</code>?</b></summary>

For object shapes they're mostly interchangeable. An `interface` can be extended with `extends` and reopened through declaration merging, which is useful for library types. A `type` alias can express things an interface can't: unions, intersections, tuples, primitives, and mapped or conditional types. What matters most is consistency within a codebase. Many teams use `type` everywhere and `interface` only when they need declaration merging.
</details>

<details>
<summary><b>26. <code>any</code> vs <code>unknown</code> vs <code>never</code>?</b></summary>

`any` turns type checking off: anything goes, and errors spread silently. `unknown` also accepts any value but forces you to narrow it before use, which makes it the safe choice for data you don't trust yet, like `JSON.parse` output or a caught error. `never` is the type with no values. It's used for functions that always throw, and for exhaustive checks: if you add a new member to a union and forget to handle it, assigning it to `never` fails at compile time.
</details>

<details>
<summary><b>27. What is narrowing? What's a discriminated union?</b></summary>

Narrowing is TypeScript refining a type inside a branch based on a check: `typeof`, `instanceof`, `in`, equality or truthiness. A discriminated union is a union where every variant has a common literal field, like `status` or `kind`, that tells them apart:

```ts
type Result =
  | { status: 'ok'; invoice: Invoice }
  | { status: 'error'; message: string };

function render(result: Result) {
  if (result.status === 'ok') return result.invoice.number; // TS knows `invoice` exists
  return result.message;
}
```
</details>

<details>
<summary><b>28. What are generics, and why use them?</b></summary>

Generics are type parameters. They let a function or a type work with many types while keeping the relationship between input and output. You use them all the time: `Array<T>`, `Promise<T>`, `useState<T>`. A minimal example is `function first<T>(items: T[]): T | undefined`. Call it with `Book[]` and you get `Book | undefined`, not `any`. You can constrain them, for example `<T extends { id: string }>`, when the function needs certain properties.
</details>

<details>
<summary><b>29. Which utility types do you use most?</b></summary>

`Partial<T>` makes every property optional, which is typical for an update payload. `Pick<T, 'a' | 'b'>` keeps some properties, and `Omit<T, 'id'>` removes some, for example a create payload without the generated `id`. `Record<K, V>` types an object used as a map. `ReturnType<typeof fn>` and `Awaited<T>` extract types from functions and promises. The point is to derive types from one source of truth instead of copying them.
</details>

<details>
<summary><b>30. How do you type data from an API you don't control?</b></summary>

`const books: Book[] = await res.json()` is a lie the compiler accepts, because `json()` returns `any`. I treat the response as `unknown` and validate it with a schema, for example zod: `const books = BookSchema.array().parse(data)`. That throws if the shape is wrong. Then I derive the TypeScript type from the schema with `z.infer<typeof BookSchema>`, so the type and the validation can never drift apart. For my own backend, types can be shared or generated from an OpenAPI spec.

_De ce se întreabă:_ aici se vede dacă ai înțeles întrebarea 24.
</details>

<details>
<summary><b>31. What does <code>strict: true</code> enable?</b></summary>

It's a family of checks. The most important is `strictNullChecks`: `null` and `undefined` must be handled explicitly instead of being assignable to everything. It also enables `noImplicitAny`, `strictFunctionTypes`, `strictPropertyInitialization`, `useUnknownInCatchVariables` and a few others. New projects should always have it on. Turning it off hides exactly the bugs TypeScript exists to catch.
</details>

<details>
<summary><b>32. Why can <code>as</code> be dangerous? What's <code>satisfies</code>?</b></summary>

A type assertion with `as` tells the compiler "trust me, it's this type", with no runtime check. If you're wrong, the error appears later and somewhere else. It's acceptable when you genuinely know more than the compiler, but `as` on API data, or `as any` to silence an error, is a smell; narrowing or validation is better. `satisfies` checks that a value matches a type without widening it, so you get both the check and the precise inferred type.

_Capcana:_ când AI-ul nu reușește să repare o eroare de tip, adesea pune `as any`. La review, caută-l.
</details>

## Mai departe

- [MDN — JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [javascript.info](https://javascript.info/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
