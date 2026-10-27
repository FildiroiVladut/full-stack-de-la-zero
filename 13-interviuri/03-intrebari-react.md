# Întrebări React și Next.js

28 de întrebări: 23 de React, 5 de Next.js. Pentru un rol de frontend sau full-stack, partea de React e garantată la interviu. Partea de Next.js apare des, pentru că e cel mai cerut framework React în anunțuri.

Citește întrebarea, răspunde cu voce tare, abia apoi deschide răspunsul.

## Bazele

Detalii: [06-react/01-cum-gandeste-react.md](../06-react/01-cum-gandeste-react.md), [06-react/02-props-si-state.md](../06-react/02-props-si-state.md), [06-react/03-liste-conditii-formulare.md](../06-react/03-liste-conditii-formulare.md)

<details>
<summary><b>1. What is React, and why use it instead of manipulating the DOM directly?</b></summary>

React is a library for building UIs out of components. Its core idea is that the UI is a function of state: you describe what the screen should look like for the current data, and React works out the DOM changes. With manual DOM manipulation, you have to keep the screen in sync with the data yourself at every change, and in a growing app that's where most bugs come from. React also gives you composition, meaning small reusable components, and a huge ecosystem.
</details>

<details>
<summary><b>2. What is JSX?</b></summary>

JSX is a syntax extension that lets you write HTML-like markup inside JavaScript. A compiler turns it into function calls that create React elements, which are plain objects describing the UI. Because it's JavaScript, you can put expressions in curly braces, map over arrays to render lists, and use conditionals. A few differences from HTML: `className` instead of `class`, camelCase attributes like `onClick`, and every element must be closed.
</details>

<details>
<summary><b>3. Props vs state?</b></summary>

Props are inputs passed from a parent to a child. They're read-only for the child, like function arguments. State is data a component owns and can change over time with a setter such as the one from `useState`. Changing state triggers a re-render. A good rule: if a value can be computed from props or other state, it shouldn't be state; compute it during render.
</details>

<details>
<summary><b>4. What causes a component to re-render?</b></summary>

Three things: its own state changes, its parent re-renders, or a context it consumes changes. By default, when a parent re-renders, all its children re-render too, even if their props are the same, unless they're wrapped in `memo`. A re-render doesn't necessarily mean a DOM update: React compares the new output with the previous one and touches only what changed.

_Capcana:_ răspunsul „când se schimbă props-urile” e doar pe jumătate corect. Props-urile se schimbă doar pentru că părintele s-a re-randat.
</details>

<details>
<summary><b>5. Why do list items need a <code>key</code>? Why not use the index?</b></summary>

Keys let React match items between renders, so it knows which item was added, removed or moved. With the array index as key, inserting, deleting or reordering items shifts the indexes, and React attaches state, like an input's value or a checkbox, to the wrong item. Use a stable unique ID from the data. The index is acceptable only for static lists that never change order.
</details>

<details>
<summary><b>6. Controlled vs uncontrolled inputs?</b></summary>

A controlled input gets its value from React state and updates that state on every change. React is the source of truth, so you can validate, format or disable things as the user types. An uncontrolled input keeps its own value in the DOM, and you read it when you need it, through a ref or `FormData` on submit. Controlled inputs are more common in forms with live validation. Uncontrolled inputs are simpler and work well with form libraries and React 19 form actions.
</details>

<details>
<summary><b>7. How do you update state that holds an object or an array?</b></summary>

You never mutate it; you create a new object or array with the change, because React compares by reference. To add an item: `setBooks([...books, newBook])`. To update one: `setBooks(books.map(b => b.id === id ? { ...b, rating } : b))`. To remove: `setBooks(books.filter(b => b.id !== id))`. If you do `books.push(newBook); setBooks(books)`, the reference is the same and React may not re-render.
</details>

<details>
<summary><b>8. Why does reading state right after <code>setState</code> give the old value?</b></summary>

Each render sees a snapshot of state. Calling the setter doesn't change the variable in the current render; it schedules a new render with the new value. React also batches several updates from the same event into one render. That's why `setCount(count + 1)` called three times in one click adds only 1. When the new value depends on the old one, use the updater form: `setCount(c => c + 1)`.
</details>

<details>
<summary><b>9. What is lifting state up? When do you reach for Context?</b></summary>

When two components need the same state, you move it to their closest common parent and pass it down as props, together with the functions that change it. Passing props through many intermediate layers becomes "prop drilling". Context solves that for values many components need, such as the current user, the theme or the language. It's not a replacement for all state: every consumer re-renders when the context value changes. For server data, use a data library; for complex client state, a small store like Zustand.
</details>

## Hooks

Detalii: [06-react/04-efecte.md](../06-react/04-efecte.md), [06-react/05-hooks-si-compozitie.md](../06-react/05-hooks-si-compozitie.md)

<details>
<summary><b>10. What is <code>useEffect</code> for, and when don't you need it?</b></summary>

`useEffect` synchronizes a component with something outside React: a subscription, a timer, a browser API, a non-React widget, or a data fetch when you have no data library. You don't need it to compute values from props or state; do that during render. You don't need it to respond to user actions either; do that in the event handler. And you don't need it to reset state when a prop changes; give the component a `key` instead. Unnecessary effects cause extra renders and subtle bugs.

_De ce se întreabă:_ folosirea greșită a lui `useEffect` e cel mai frecvent semn de cod React scris de un junior sau generat de AI.
</details>

<details>
<summary><b>11. Explain the dependency array and the cleanup function.</b></summary>

With no array, the effect runs after every render. With `[]`, it runs after the first render, the mount. With `[a, b]`, it runs after renders where `a` or `b` changed, compared with `Object.is`. The function you return is the cleanup: React runs it before the next run of the effect and when the component unmounts. That's where you unsubscribe, clear timers or abort requests.
</details>

<details>
<summary><b>12. Why does my effect run twice in development?</b></summary>

Because of `StrictMode`. In development, React mounts each component, unmounts it and mounts it again, to check that your effects clean up properly. It also calls render functions twice to catch impure code. In production it runs once. The fix isn't removing `StrictMode`; it's writing a proper cleanup, so that mount, cleanup, mount behaves the same as a single mount.
</details>

<details>
<summary><b>13. What are the rules of hooks, and why do they exist?</b></summary>

Call hooks only at the top level of a component or custom hook: not inside conditions, loops or after an early return. And call them only from React function components or custom hooks. React identifies each hook by the order of calls, so if that order changes between renders, state gets mixed up. The ESLint plugin for hooks enforces the rules. The React 19 `use` API is an exception and can be called conditionally.
</details>

<details>
<summary><b>14. What is <code>useRef</code> used for?</b></summary>

Two things. First, holding a reference to a DOM element, to focus an input, measure something or scroll. Second, keeping a mutable value that survives re-renders without causing one, like a timer ID or the previous value of a prop. Changing `ref.current` doesn't trigger a render. You shouldn't read or write it during render, only in effects and event handlers.
</details>

<details>
<summary><b>15. When are <code>useMemo</code> and <code>useCallback</code> worth it?</b></summary>

`useMemo` caches a computed value and `useCallback` caches a function between renders. They're worth it when a computation is genuinely expensive, or when you need a stable reference: a prop passed to a `memo` component, or a dependency of an effect. Everywhere else they add noise and cost something themselves. I measure first with the React DevTools Profiler. The React Compiler can add this memoization automatically, which reduces the need to write it by hand.
</details>

<details>
<summary><b>16. What is a custom hook? Give an example.</b></summary>

A custom hook is a function whose name starts with `use` and that calls other hooks. It extracts stateful logic so several components can reuse it. Each component that uses it gets its own state; the logic is shared, not the data. A typical example is `useDebouncedValue(value, delay)`, which returns the value only after the user stops typing for `delay` milliseconds. In Shelf, the search box uses exactly that.
</details>

<details>
<summary><b>17. What is a stale closure in React?</b></summary>

A function created during a render captures the props and state of that render. If it keeps running later, for example in a `setInterval` started in an effect with `[]`, it keeps seeing the old values forever. Fixes: use the updater form of the setter (`setCount(c => c + 1)`), list the right dependencies so the effect is recreated, or keep the latest value in a ref.
</details>

## Date și arhitectură

Detalii: [06-react/06-routing-si-date.md](../06-react/06-routing-si-date.md)

<details>
<summary><b>18. How do you fetch data in React? Why use TanStack Query instead of <code>useEffect</code> + <code>fetch</code>?</b></summary>

With `useEffect` and `fetch`, you have to handle loading and error states, race conditions when inputs change quickly, caching, refetching, retries and deduplication yourself, and most hand-written versions miss some of these. TanStack Query handles them: you give it a query key and a function that returns a promise, and you get `data`, `isPending` and `error`, plus caching, background refetching and cache invalidation after mutations. In Next.js, a lot of data fetching moves to Server Components instead.
</details>

<details>
<summary><b>19. Where do you keep different kinds of state?</b></summary>

It depends on the kind. Server state, meaning data that lives in the backend, goes in TanStack Query's cache. State that should be shareable or bookmarkable, like filters, search and the current page, goes in the URL's search params. Local UI state, like a modal being open, goes in `useState` in the component that needs it. Client state shared by many components goes in Context or a small store. Form state goes in controlled inputs or a form library. Mixing these up, for example copying server data into `useState`, is a classic source of bugs.
</details>

<details>
<summary><b>20. How does client-side routing work?</b></summary>

The router uses the browser's History API to change the URL without reloading the page, and renders the component that matches the new URL. The whole app is one HTML page with a JavaScript bundle. That's why a static host needs a rewrite rule sending every path to `index.html`: otherwise refreshing on `/books/42` returns a 404, because there's no file at that path on the server.
</details>

<details>
<summary><b>21. How do you handle forms and validation?</b></summary>

For simple forms, controlled inputs or reading `FormData` on submit; for bigger ones, a library like React Hook Form. I define the validation rules once in a schema, for example with zod, and use it for the form. Client-side validation is for user experience. The backend must validate everything again, because anyone can call the API directly. React 19 also adds form actions: `<form action={fn}>` together with `useActionState` for pending state and errors.
</details>

<details>
<summary><b>22. A page feels slow. How do you find and fix the problem?</b></summary>

Measure first: the React DevTools Profiler shows which components re-render and how long they take, and the browser's Performance tab shows the rest. Common fixes: move state down so fewer components re-render; avoid creating new objects as props for memoized children; virtualize long lists so only the visible rows render; split code with `lazy` so the first load is smaller; debounce expensive work triggered by typing. Don't add `useMemo` everywhere without measuring.
</details>

<details>
<summary><b>23. What's new in React 19?</b></summary>

The main theme is Actions: async functions for data mutations, which you can pass to `<form action>`. They come with `useActionState` for pending and error state, `useOptimistic` for optimistic updates, and `useFormStatus` for the form's status. There's also the `use` API for reading a promise or a context. `ref` can now be passed as a normal prop, so `forwardRef` is no longer needed. React 19 also stabilized Server Components, which frameworks like Next.js use.
</details>

## Next.js

Detalii: [06-react/08-nextjs.md](../06-react/08-nextjs.md)

<details>
<summary><b>24. What does Next.js add over React with Vite?</b></summary>

React with Vite gives you a client-side single-page app, and you need a separate backend for anything secret or database-related. Next.js is a full framework on top of React: file-based routing, server rendering (per request, at build time, or streamed), Server Components, Route Handlers for API endpoints, Server Actions, plus image and font optimization. Frontend and backend code live in one project. In my projects, Shelf, Invoicer and Booking use Vite with a separate NestJS API, and Kitchen AI uses Next.js, so I've worked with both architectures.
</details>

<details>
<summary><b>25. Server Components vs Client Components?</b></summary>

In the App Router, components are Server Components by default. They render only on the server, their code is never sent to the browser, they can be `async`, and they can read the database or use secret keys directly. They can't use state, effects or browser APIs. A file that starts with `'use client'` defines Client Components. Those are also pre-rendered on the server, then hydrated in the browser, so they can use hooks and event handlers. Everything a client file imports becomes part of the client bundle, so I keep the `'use client'` boundary as low in the tree as possible.
</details>

<details>
<summary><b>26. In Next.js, where can you safely use a secret API key?</b></summary>

Only in server code: Server Components, Route Handlers and Server Actions. Environment variables without the `NEXT_PUBLIC_` prefix are available only on the server. Anything prefixed with `NEXT_PUBLIC_` is inlined into the JavaScript bundle at build time, so anyone can read it in DevTools. As an extra guard, I import the `server-only` package in modules that use secrets, so the build fails if one gets imported into client code.

_De ce se întreabă:_ e greșeala cu cele mai mari consecințe, iar intervievatorii știu că AI-ul o face des.
</details>

<details>
<summary><b>27. CSR vs SSR vs SSG?</b></summary>

With CSR, client-side rendering, the browser gets a nearly empty HTML page plus JavaScript and builds the UI itself. That's a classic SPA. With SSR, server-side rendering, the server builds the HTML on every request, which is good for dynamic, personalized or SEO-sensitive pages. With SSG, static generation, the HTML is built once at build time; it's the fastest option, good for content that rarely changes. Next.js lets you choose per route and also revalidate static pages periodically, which sits in between.
</details>

<details>
<summary><b>28. What are Server Actions, and what's the security catch?</b></summary>

Server Actions are async functions marked with `'use server'`. You can call them from a form or a client component, and they run on the server. It feels like calling a local function, but Next.js exposes each one as an HTTP endpoint that anyone can call with their own payload. So every action must validate its input, for example with zod, and check authentication and authorization inside itself, exactly like an API route.
</details>

## Mai departe

- [react.dev — Learn](https://react.dev/learn)
- [react.dev — You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Next.js Docs](https://nextjs.org/docs)
