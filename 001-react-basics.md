# ⚛️ React Basics

A practical reference guide for learning React from scratch — covering JSX, props, state, hooks, and context, built hands-on inside this Vite playground.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [Why a Framework?](#why-a-framework)
  - [A Short History of React](#a-short-history-of-react)
  - [What Is a Component?](#what-is-a-component)
- [Project Setup](#-project-setup)
  - [Creating the Project](#creating-the-project)
  - [Keeping React Up to Date](#keeping-react-up-to-date)
  - [Multi-Page Setup with Vite](#multi-page-setup-with-vite)
  - [Dev, Build & Preview](#dev-build--preview)
- [Hello World & JSX](#-hello-world--jsx)
  - [Creating a Component](#creating-a-component)
  - [Rendering with createRoot & StrictMode](#rendering-with-createroot--strictmode)
  - [JSX Rules](#jsx-rules)
  - [JavaScript Inside JSX](#javascript-inside-jsx)
- [Composing Components](#-composing-components)
  - [Splitting into Smaller Components](#splitting-into-smaller-components)
  - [Nested Components with children](#nested-components-with-children)
  - [Styling with className](#styling-with-classname)
- [Props](#-props)
  - [Adding Props](#adding-props)
  - [Destructuring & Default Values](#destructuring--default-values)
  - [Spread Syntax](#spread-syntax)
- [Conditional Rendering](#-conditional-rendering)
  - [Returning null](#returning-null)
  - [Ternary Operator](#ternary-operator)
- [Rendering Collections & Keys](#-rendering-collections--keys)
  - [Mapping an Array to Components](#mapping-an-array-to-components)
  - [Why Keys Matter](#why-keys-matter)
- [Pure Components](#-pure-components)
  - [Pure Functions](#pure-functions)
  - [An Impure vs. a Pure Component](#an-impure-vs-a-pure-component)
- [Event Handling](#-event-handling)
  - [Adding Event Handlers](#adding-event-handlers)
  - [Event Handlers as Props](#event-handlers-as-props)
  - [The Event Object](#the-event-object)
  - [Event Propagation](#event-propagation)
  - [Preventing Default Behavior](#preventing-default-behavior)
- [Side Effects & DOM Manipulation](#-side-effects--dom-manipulation)
  - [Manual DOM Manipulation](#manual-dom-manipulation)
  - [Introducing Hooks](#introducing-hooks)
- [State with useState](#-state-with-usestate)
  - [Why a Local Variable Doesn't Work](#why-a-local-variable-doesnt-work)
  - [The useState Hook](#the-usestate-hook)
  - [State Is Isolated & Private](#state-is-isolated--private)
- [Render Lifecycle & State Snapshots](#-render-lifecycle--state-snapshots)
  - [Trigger, Render, Commit](#trigger-render-commit)
  - [State as a Snapshot](#state-as-a-snapshot)
  - [Batching Multiple Updates](#batching-multiple-updates)
  - [Updater Functions](#updater-functions)
- [Immutable State: Objects & Arrays](#-immutable-state-objects--arrays)
  - [Updating Objects with Spread Syntax](#updating-objects-with-spread-syntax)
  - [Immer & use-immer](#immer--use-immer)
  - [Updating Arrays Immutably](#updating-arrays-immutably)
- [Sharing State Between Components](#-sharing-state-between-components)
  - [Lifting State Up](#lifting-state-up)
- [State, Position & Resetting with key](#-state-position--resetting-with-key)
  - [State Persists by Position](#state-persists-by-position)
  - [Resetting State with the key Prop](#resetting-state-with-the-key-prop)
- [useReducer for Complex State](#-usereducer-for-complex-state)
  - [Why a Reducer](#why-a-reducer)
  - [Writing a Reducer Function](#writing-a-reducer-function)
  - [useImmerReducer](#useimmerreducer)
- [Context](#-context)
  - [Creating & Providing a Context](#creating--providing-a-context)
  - [Reading Context with useContext](#reading-context-with-usecontext)
  - [Combining Context with a Reducer](#combining-context-with-a-reducer)
- [useRef](#-useref)
  - [Ref vs. State](#ref-vs-state)
  - [A Timer Built with useRef](#a-timer-built-with-useref)
  - [Manipulating the DOM with ref](#manipulating-the-dom-with-ref)
- [useEffect](#-useeffect)
  - [Why Event Handlers Aren't Enough](#why-event-handlers-arent-enough)
  - [Fetching Data in an Effect](#fetching-data-in-an-effect)
  - [Effect Dependencies](#effect-dependencies)
  - [Async Code Inside an Effect](#async-code-inside-an-effect)
  - [Effect Cleanup](#effect-cleanup)
  - [When Not to Use an Effect](#when-not-to-use-an-effect)
- [useMemo](#-usememo)
  - [Memoizing an Expensive Calculation](#memoizing-an-expensive-calculation)
- [Custom Hooks](#-custom-hooks)
  - [Extracting useOnline()](#extracting-useonline)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### Why a Framework?

- A framework standardizes how a frontend project is built — especially valuable in a team, where everyone would otherwise write code in their own style
- Without a framework, every developer ends up with a different structure, making a codebase much harder to maintain

### A Short History of React

- React started life as **FaxJS**, developed inside Facebook around 2010 to solve feed/timeline updates without a full page refresh
- In 2011, FaxJS was integrated with XHP (Facebook's PHP framework) and renamed **ReactJS**
- In 2013, Facebook open-sourced ReactJS at a JavaScript conference
- Today, React (as it's commonly called) is one of the most popular frontend frameworks

> Reference: [github.com/facebook/react](https://github.com/facebook/react) · [react.dev](https://react.dev/)

### What Is a Component?

- A **Component** is a piece of code that can be used independently — it can contain one or more HTML elements, JavaScript, and CSS
- There's no rule about how big or small a Component should be — think of it like a function: it can be large or small, as long as it's reusable
- Component structure mirrors the DOM (Document Object Model): a Component can contain other Components inside it

---

## 🏗️ Project Setup

> **Key Insight:** This isn't a single-page app with client-side routing — it's a genuine **multi-page Vite app**. Every feature (`counter.html`, `note.html`, `profile.html`, …) is its own independent HTML entry point, each wired to its own `main.jsx`, registered explicitly in `vite.config.js`.

### Creating the Project

Vite is used as the build tool. To scaffold a React project:

```bash
# JavaScript template
npm create vite@latest nama-aplikasi -- --template react

# TypeScript template
npm create vite@latest nama-aplikasi -- --template react-ts
```

> Reference: [vite.dev/guide/#scaffolding-your-first-vite-project](https://vite.dev/guide/#scaffolding-your-first-vite-project)

### Keeping React Up to Date

Always make sure `react` and `react-dom` are on the latest version.

> Reference: [npmjs.com/package/react](https://www.npmjs.com/package/react) · [npmjs.com/package/react-dom](https://www.npmjs.com/package/react-dom)

### Multi-Page Setup with Vite

Each feature area in this repo (`hello-world`, `task`, `counter`, `note`, `timer`, `guestbook`, `product`, `online`, `profile`, …) has its own `<name>.html` entry point at the project root, pointing at its own `src/<name>/main.jsx`.

**`hello-world.html`**

```html
<!doctype html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>study-basic-react</title>
</head>

<body>
  <div id="root"></div>
  <script type="module" src="src/hello-world/main.jsx"></script>
</body>

</html>
```

Every new page has to be registered as a build input in `vite.config.js`, or `vite build` won't emit it:

**`vite.config.js`**

```javascript
import {defineConfig} from "vite";
import react from "@vitejs/plugin-react";

// https://vite.dev/config/
export default defineConfig({
    plugins: [react()],
    build: {
        rollupOptions: {
            input: {
                index: "index.html",
                hello_world: "hello-world.html",
                contact: "contact.html",
                task: "task.html",
                counter: "counter.html",
                note: "note.html",
                timer: "timer.html",
                guestbook: "guestbook.html",
                product: "product.html",
                online: "online.html",
            },
        },
    },
});
```

### Dev, Build & Preview

```bash
npm run dev      # start the dev server
npm run build    # build every registered page into dist/
npm run preview  # preview the production build
```

---

## 👋 Hello World & JSX

### Creating a Component

A Component usually lives in its own `.jsx` file, named after the Component. The default export is a function whose return value is the UI to render.

**`src/hello-world/HelloWorld.jsx`**

```jsx
function HeaderHelloWorld({ text = "Ups, I forgot include the text" }) {
  return (
    <h1 className="title">
      {text.toUpperCase()}
    </h1>
  )
}
```

### Rendering with createRoot & StrictMode

To display a Component, you need a React root — created with `createRoot(element)` — and then `render(component)` to mount it. `StrictMode` wraps the root to help surface bugs during development.

> Reference: [react.dev/reference/react-dom/client/createRoot](https://react.dev/reference/react-dom/client/createRoot) · [react.dev/reference/react/StrictMode](https://react.dev/reference/react/StrictMode)

**`src/hello-world/main.jsx`**

```jsx
import { createRoot } from 'react-dom/client'
import { StrictMode } from "react"
import HelloWorld from "./HelloWorld.jsx"
import Container from "./Container.jsx"

createRoot(
  document.getElementById('root')
).render(
  <StrictMode>
    <Container>
      <HelloWorld />
    </Container>
  </StrictMode>
)
```

### JSX Rules

Converting existing HTML to JSX isn't a copy-paste job — JSX is stricter than HTML:

- A Component may only return **one** element — wrap multiple elements in a parent element
- Every tag must be **closed**, including self-closing ones (`<img />`, not `<img>`)
- Attributes use **camelCase**, because JSX attributes become JavaScript variable names — `className` instead of `class`, `strip-case` isn't valid JavaScript

If you already have HTML and want to convert it, use a converter instead of doing it by hand.

> Reference: [transform.tools/html-to-jsx](https://transform.tools/html-to-jsx)

### JavaScript Inside JSX

Curly braces `{ }` let you drop JavaScript into JSX in exactly two places: as text content, or as an attribute value (e.g. `<img src={location} />`). A `style` attribute takes a plain JavaScript object, which is why it often looks like a "double brace" — it's really just one brace holding an object literal.

**`src/hello-world/HelloWorld.jsx`**

```jsx
import "./HelloWorld.css"

function HeaderHelloWorld({ text = "Ups, I forgot include the text" }) {
  return (
    <h1 className="title">
      {text.toUpperCase()}
    </h1>
  )
}

function ContentHelloWorld() {
  const text = "Happy Learning Basic ReactJS"
  return (
    <p className="content">
      {text}
    </p>
  )
}

function HelloWorld() {
  const props = {
    text: "Hello World"
  }

  return (
    <div>
      <HeaderHelloWorld {...props} />
      <ContentHelloWorld />
    </div>
  );
}

export default HelloWorld;
```

---

## 🧱 Composing Components

### Splitting into Smaller Components

A Component is really just a function that returns a React element — so splitting one big Component into smaller ones is as simple as writing more functions that each return part of the UI, then composing them.

### Nested Components with children

JSX supports nested composition: a Component can be built to hold other Components dynamically via the special `children` prop.

**`src/hello-world/Container.jsx`**

```jsx
export default function Container({ children }) {
  return (
    <div>
      <h1>Header Example</h1>
      {children}
      <footer>
        <p>2026 Footer Example</p>
      </footer>
    </div>
  )
}
```

### Styling with className

CSS isn't part of JSX — it lives in its own `.css` file (or inline, as a style object). JSX elements use `className`, not `class`.

> Reference: [react.dev/reference/react-dom/components/common#applying-css-styles](https://react.dev/reference/react-dom/components/common#applying-css-styles)

**`src/hello-world/HelloWorld.css`**

```css
.title {
  color: red;
  background-color: aqua;
}

.content {
  color: blue;
  background-color: yellow;
}
```

---

## 📦 Props

React Components communicate through **Props** — a parent Component sends information down to a child Component with them. Props behave like HTML attributes, except you can pass any JavaScript value: objects, arrays, functions, anything.

### Adding Props

To add Props to a Component, add an object parameter to its function — that parameter is a plain JavaScript object, so you can read whatever attributes the parent sent.

### Destructuring & Default Values

Destructuring the Props parameter is common practice — it documents which attributes a Component expects, right in the signature. Because Props is a plain object, you can also give attributes default values.

**`src/button/MyButton.jsx`**

```jsx
export default function MyButton({ text, onSmash }) {

  return (
    <button onClick={onSmash}>{text}</button>
  )
}
```

### Spread Syntax

Forwarding every prop from a parent to a child individually gets tedious fast — JavaScript's spread syntax solves that.

**`src/hello-world/main.jsx`** (excerpt)

```jsx
function HelloWorld() {
  const props = {
    text: "Hello World"
  }

  return (
    <div>
      <HeaderHelloWorld {...props} />
      <ContentHelloWorld />
    </div>
  );
}
```

---

## 🔀 Conditional Rendering

Different UI states usually call for different output. JSX handles this with plain JavaScript.

### Returning null

Returning `null` tells React to render nothing for that Component.

**`src/todolist/Todo.jsx`**

```jsx
export default function Todo({ text, isCompleted = false, isDeleted = false }) {
  if (isDeleted) {
    return null
  } else {
    return (
      <div>
        <li>{text} {isCompleted && "✅"}</li>
      </div>
    )
  }
}
```

### Ternary Operator

For simpler cases, JavaScript's ternary operator works directly inside JSX.

---

## 📋 Rendering Collections & Keys

JSX has no built-in loop construct. To render a Component once per item in a collection, reach for JavaScript's `Array.prototype.map()`.

### Mapping an Array to Components

**`src/todolist/Todolist.jsx`**

```jsx
import Todo from "./Todo";

export default function Todolist() {
  const data = [
    {
      id: 1,
      text: "Learn HTML",
      isCompleted: true,
      isDeleted: true
    },
    {
      id: 2,
      text: "Learn CSS",
      isCompleted: true,
      isDeleted: false
    },
    {
      id: 3,
      text: "Learn JS",
      isCompleted: true,
      isDeleted: false
    },
    {
      id: 4,
      text: "Learn React",
      isCompleted: false,
      isDeleted: false
    }
  ]

  return (
    <ul>
      {data.map((todo) => (
        <Todo {...todo} key={todo.id}/>
      ))}
    </ul>
  )
}
```

### Why Keys Matter

A missing `key` prop shows up as an editor warning: *"Missing key prop for element in iterator."* Every Component in a collection needs a unique `key` (string or number) so React can track it as items are reordered, removed, or added — usually pulled straight from the data, so it stays stable.

---

## 🧼 Pure Components

### Pure Functions

A function is **pure** when it meets two criteria: it always returns the same output for the same input, and it has no side effects (it doesn't mutate anything outside its own scope).

> Reference: [Pure function (Wikipedia)](https://en.wikipedia.org/wiki/Pure_function)

> **Key Insight:** React assumes every Component you write is a pure function. It's technically possible to write an impure Component, but strongly discouraged — calling it with the same input can then produce inconsistent output.

### An Impure vs. a Pure Component

An impure `Row` mutates a variable declared outside the Component every time it renders:

```jsx
// ❌ Not a pure component
let counter = 0

export default function Row({ text }) {
  counter++
  return (
    <tr>
      <td>{counter}</td>
      <td>{text}</td>
    </tr>
  )
}
```

The actual, pure version in this repo derives everything from its own props instead:

**`src/table/Row.jsx`**

```jsx
export default function Row({ no, text }) {
  return (
    <tr>
      <td>{no}</td>
      <td>{text}</td>
    </tr>
  )
}
```

**`src/table/Table.jsx`**

```jsx
import Row from "./Row.jsx"

export default function Table() {
  return (
    <table>
      <thead>
        <tr>
          <th>No</th>
          <th>Text</th>
        </tr>
      </thead>
      <tbody>
        <Row no={1} text="One" />
        <Row no={2} text="Two" />
        <Row no={3} text="Three" />
      </tbody>
    </table>
  )
}
```

---

## 🖱️ Event Handling

React elements support the same event handlers as HTML elements.

> Reference: [react.dev/reference/react-dom/components/common](https://react.dev/reference/react-dom/components/common)

### Adding Event Handlers

Handlers are usually named starting with `handle`, followed by the event, e.g. `handleClick`. Defining the handler function inside the Component also gives it access to that Component's props.

**`src/button/AlertButton.jsx`**

```jsx
import {useRef} from "react";

export default function AlertButton({text, message}) {
    const counter = useRef(0)

    function handleClick(e) {
        console.info(e.target)
        alert(`${message} ${counter.current++}`)
    }

    return (
        <button onClick={handleClick}>{text}</button>
    )
}
```

*(`counter` here is a `useRef` — a way to remember a value across clicks without triggering a re-render; see [useRef](#-useref) below.)*

### Event Handlers as Props

Because Props is just a JavaScript object, an attribute in Props can hold a function — including an event handler. By convention, a Props attribute that holds an event handler is named starting with `on`, e.g. `onSmash`.

**`src/button/MyButton.jsx`**

```jsx
export default function MyButton({ text, onSmash }) {

  return (
    <button onClick={onSmash}>{text}</button>
  )
}
```

### The Event Object

A handler function can take a React event object as its parameter — it's compatible with the standard DOM Event object.

> Reference: [react.dev/reference/react-dom/components/common#react-event-object](https://react.dev/reference/react-dom/components/common#react-event-object) · [MDN: Event](https://developer.mozilla.org/en-US/docs/Web/API/Event)

### Event Propagation

Events in a React tree always propagate upward to parent elements. If a `div` and a `button` inside it both have `onClick`, clicking the button fires the button's handler *and then* the div's. Stop that with `stopPropagation()`.

> Reference: [MDN: Event.stopPropagation()](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation)

**`src/button/Toolbar.jsx`**

```jsx
export default function Toolbar({ onClick }) {
  return (
    <div onClick={onClick} style={{ backgroundColor: "yellow"}}>
      <button onClick={onClick}>First</button>
      <button onClick={onClick}>Second</button>
    </div>
  )
}
```

Used with a handler that calls `stopPropagation()`:

```jsx
<Toolbar onClick={(e) => {
  e.stopPropagation();
  alert("You click toolbar")
}} />
```

### Preventing Default Behavior

Besides stopping propagation, it's common to stop an element's default action with `preventDefault()` — e.g. stopping a form from actually submitting.

> Reference: [MDN: Event.preventDefault()](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault)

**`src/form/SearchForm.jsx`**

```jsx
export default function SearchForm() {
  return (
    <form>
      <input type="text" />
      <button
        onClick={(e) => {
          e.preventDefault();
          alert("You Search");
        }}
      >
        Cari
      </button>
    </form>
  );
}
```

---

## 🧯 Side Effects & DOM Manipulation

### Manual DOM Manipulation

Nothing stops a Component from touching the DOM directly inside an event handler — but it's discouraged, especially once several different handlers can all trigger the same change.

**`src/form/SayHelloForm.jsx`**

```jsx
export default function SayHelloForm() {
  function handleClick(e) {
    e.preventDefault();
    const name = document.getElementById("input_name").value;
    document.getElementById("text_hello").innerText = "Hello " + name;
  }
  return (
    <div>
      <form>
        <input id="input_name" />
        <button onClick={handleClick}>Say Hello</button>
      </form>
      <h1 id="text_hello">Hello World</h1>
    </div>
  );
}
```

### Introducing Hooks

The recommended replacement for manual DOM manipulation is **State** — which is one of many **Hooks** React provides.

> Reference: [react.dev/reference/react/hooks](https://react.dev/reference/react/hooks)

---

## 🧠 State with useState

A Component often needs to *remember* something between renders — the current count, the current image index, and so on. In React, a Component's private memory is called **State**.

### Why a Local Variable Doesn't Work

A plain local variable can't hold State: every time React re-renders a Component, all of its code runs again, so a local variable resets to its initial value — and changing it never triggers a re-render in the first place.

```jsx
// ❌ counter is a plain local variable — it never sticks and never re-renders
export default function Counter() {
  let counter = 0;

  function handleClick() {
    counter = counter + 1
    console.log(counter)
  }
  return (
    <>
      <button onClick={handleClick}>Increment</button>
      <h1>Counter: {counter}</h1>
    </>
  )
}
```

### The useState Hook

`useState(initial)` returns an array of two values: the state itself, and a function to update it. A Component using that state is automatically re-rendered when it changes.

> Reference: [react.dev/reference/react/useState](https://react.dev/reference/react/useState)

**`src/counter/Counter.jsx`**

```jsx
import {useState} from "react";

export default function Counter({name}) {
    const [count, setCount] = useState(0);

    function handleClick() {
        setCount(count + 1);
    }

    return (
        <>
            <h1>Counter {name}: {count}</h1>
            <button onClick={handleClick}>Increment</button>
        </>
    )
}
```

### State Is Isolated & Private

State is local to the specific Component instance using it — rendering the same Component multiple times gives each instance its own, separate State.

---

## 🔄 Render Lifecycle & State Snapshots

### Trigger, Render, Commit

Before a Component appears on screen, React runs it through three steps:

1. **Trigger** — a render is triggered either by the initial `render()` call, or by a State update
2. **Render** — React calls your Components to figure out what should be on screen. On initial render, React calls the root Component; on a re-render, only the Components whose State changed
3. **Commit** — React applies the minimal set of DOM changes needed. On the very first render there's nothing in the DOM yet, so React uses `appendChild()`; on a re-render, it only touches elements that actually differ from the previous result

### State as a Snapshot

> **Key Insight:** A State variable looks like an ordinary JavaScript variable, but it behaves like a **snapshot** of a specific render. Calling the setter doesn't mutate that snapshot — it schedules a new render with a new snapshot. The screen only appears to update "live" because a State change triggers that re-render.

```jsx
import { useState } from "react";

export default function Counter() {
  const [counter, setCounter] = useState(0)
  console.log("Render Counter", counter)

  function handleClick() {
    setCounter(counter + 1)
    setCounter(counter + 1)
    setCounter(counter + 1)
    console.log(counter)
  }
  return (
    <>
      <button onClick={handleClick}>Increment</button>
      <h1>Counter: {counter}</h1>
    </>
  )
}
```

### Batching Multiple Updates

Calling `setCounter(counter + 1)` three times in a row inside `handleClick` does **not** trigger three re-renders, and the counter does not go up by 3. Each call still reads `counter` from the *same* snapshot, so all three calls schedule "set counter to (current + 1)" using the identical starting value. React batches every State update inside one event handler and re-renders once, after the handler finishes running.

### Updater Functions

To update State based on a value that hasn't been re-rendered yet, pass an **updater function** instead of a plain value:

**`src/form/Counter.jsx`**

```jsx
import { useState } from "react";

export default function Counter() {
  const [counter, setCounter] = useState(0)

  console.log("Render Counter", counter)

  function handleClick() {
    setCounter(c => c + 1)
    setCounter(c => c + 1)
    setCounter(c => c + 1)
    console.log(counter)
  }
  return (
    <>
      <button onClick={handleClick}>Increment</button>
      <h1>Counter: {counter}</h1>
    </>
  )
}
```

Now each call receives the latest queued value (`c`) rather than the stale snapshot, so three clicks really do add three.

---

## 🧬 Immutable State: Objects & Arrays

State can hold any JavaScript value, including objects and arrays — but you're expected to treat that data as **immutable**.

> **Key Insight:** Treat State as read-only. Mutating an object or array already stored in State won't trigger a re-render (React has no way to detect the mutation), so updates must always replace State with a *new* object or array.

### Updating Objects with Spread Syntax

**`src/contact/ContactForm.jsx`**

```jsx
import {useState} from "react";
import {useImmer} from "use-immer";

const initialData = {
    name: "",
    message: ""
}
```

An object in State should be replaced with a new object via the spread operator (`{ ...contact, name: e.target.value }`) rather than mutated in place.

### Immer & use-immer

[Immer](https://github.com/immerjs/immer) makes producing an immutable update far less tedious than manual spreading, especially for deep/nested objects. [`use-immer`](https://github.com/immerjs/use-immer) wires Immer directly into React State: swap `useState()` for `useImmer()`, and update State by mutating a `draft` inside a function instead of building a whole new object by hand.

```bash
npm install immer use-immer
```

**`src/contact/ContactForm.jsx`**

```jsx
import {useState} from "react";
import {useImmer} from "use-immer";

const initialData = {
    name: "",
    message: ""
}

export default function ContactForm() {
    const [contact, setContact] = useImmer(initialData)

    function handleNameChange(e) {
        setContact(
            draft => draft.name = e.target.value
        )
    }

    function handleMessageChange(e) {
        setContact(draft => draft.message = e.target.value)
    }

    return (
        <div>
            <h1>Contact Form</h1>
            <form>
                <input type="text" placeholder="Name" value={contact.name} onChange={handleNameChange}/>
                <br/>
                <input type="text" placeholder="Message" value={contact.message} onChange={handleMessageChange}/>
            </form>
            <h1>Contact Detail</h1>
            <p>
                Name: {contact.name}
            </p>
            <p>
                Message: {contact.message}
            </p>
        </div>
    )
}
```

### Updating Arrays Immutably

Arrays in State need the same immutable treatment:

| Operation | Avoid | Use Instead |
| --- | --- | --- |
| Add | `push`, `unshift` | spread syntax (`[...arr, item]`) |
| Remove | `pop`, `shift`, `splice` | `filter`, `slice` |
| Change | `splice`, `arr[i] = ...` | `map` |
| Sort | `reverse`, `sort` | build a new array |

**`src/task/Task.jsx`** shows this with `use-immer`, pushing into a `draft` array instead of mutating State directly:

```jsx
import TaskForm from "./TaskForm.jsx";
import {useImmer} from "use-immer";
import TaskList from "./TaskList.jsx";

export default function Task() {
    const [items, setItems] = useImmer([]);

    function handleSubmit(item) {
        setItems((draft) => {
            draft.push(item);
        })
    }

    return (
        <>
            <TaskForm onSubmit={handleSubmit}/>
            <TaskList items={items}/>
        </>
    );
}
```

---

## 🤝 Sharing State Between Components

### Lifting State Up

When several Components need to change together, move their State up to the closest shared parent, then pass it back down through Props. The Task page splits into a form Component and a list Component, with `Task` owning the shared `items` State.

**`src/task/TaskForm.jsx`**

```jsx
import {useState} from "react";

export default function TaskForm({onSubmit}) {
    const [item, setItem] = useState("")

    function handleChange(e) {
        setItem(e.target.value)
    }

    function handleClick(e) {
        e.preventDefault();
        onSubmit(item)
    }

    return (
        <>
            <h1>Create Task</h1>
            <form>
                <input value={item} onChange={handleChange}/>
                <button onClick={handleClick}>Add</button>
            </form>
        </>
    );
}
```

**`src/task/TaskList.jsx`**

```jsx
export default function TaskList({items = []}) {
    return (
        <>
            <h1>List Task</h1>
            <ul>
                {items.map((item, index) =>
                    <li key={index}>{item}</li>
                )}
            </ul>
        </>
    )
}
```

---

## 📍 State, Position & Resetting with key

### State Persists by Position

State isn't stored inside the Component itself — it's stored by React, tied to *where* that Component sits in the UI tree. If a Component of the same type is rendered at the same position, React keeps its State. Remove it from the tree (or swap it for a different Component/position), and its State is discarded.

### Resetting State with the key Prop

Sometimes you *want* two instances of the same Component at the same position to be treated as genuinely separate — the standard way to force that is to give each one a distinct `key`. Different keys tell React "these are different Components," so each gets fresh, independent State instead of sharing one.

**`src/counter/CounterApp.jsx`**

```jsx
import {useState} from "react";
import Counter from "./Counter.jsx";

export default function CounterApp() {
    const [show2, setShow2] = useState(true)

    function handleChange(e) {
        setShow2(e.target.checked)
    }

    return (
        <div>
            {show2 ? <Counter name="Rizky" key="rizky"/> : <Counter name="Dzaru" key="dzaru"/>}
            <div>
                <input type="checkbox" checked={show2} onChange={handleChange}/>
                Show Counter 2
            </div>
        </div>
    )
}
```

Without the `key` props here, toggling the checkbox would swap the *name* shown but keep the same underlying `count` State, since both branches render a `Counter` at the same tree position.

---

## 🧮 useReducer for Complex State

### Why a Reducer

When a Component has many different State-updating actions, keeping a separate handler for each one gets hard to maintain. A **reducer** consolidates all of that update logic into one function outside the Component. Instead of telling React *how* to change State directly, you dispatch an **action** describing *what the user did*, and the reducer decides what the next State should be.

> Reference: [react.dev/reference/react/useReducer](https://react.dev/reference/react/useReducer)

### Writing a Reducer Function

A reducer function takes the current State (or, with Immer, a mutable `draft`) and an `action` object, and produces the next State.

**`src/note/NoteApp.jsx`** (reducer excerpt)

```jsx
function notesReducer(draft, action) {
    if (action.type === "ADD_NOTE") {
        draft.push({
            id: id++,
            text: action.text,
            done: false
        })
    } else if (action.type === "DELETE_NOTE") {
        const index = draft.findIndex(note => note.id === action.id)
        draft.splice(index, 1)
    } else if (action.type === "CHANGE_NOTE") {
        const index = draft.findIndex(note => note.id === action.id)
        draft[index].done = action.done
        draft[index].text = action.text
    }
}
```

Dispatching an action just describes intent — the reducer owns the "how":

```jsx
dispatch({
    type: "ADD_NOTE",
    text: text,
})
```

### useImmerReducer

`use-immer` also supports reducers via `useImmerReducer()`, so the reducer body can mutate a `draft` directly instead of constructing a brand-new State object by hand — exactly what `notesReducer` above does.

---

## 🌐 Context

Passing information through many layers of Props ("prop drilling") gets unwieldy fast — especially for a single value many Components need. **Context** lets a parent Component provide a value that *any* Component below it can read, no matter how deeply nested. It's often described as a form of global state.

### Creating & Providing a Context

Create a context with `createContext()`, and make it available to descendants with its `Provider`.

> Reference: [react.dev/reference/react/createContext](https://react.dev/reference/react/createContext) · [Provider](https://react.dev/reference/react/createContext#provider)

**`src/profile/ProfileContext.jsx`**

```jsx
import {createContext} from "react";

export const ProfileContext = createContext();
```

**`src/profile/ProfileApp.jsx`**

```jsx
import {ProfileContext} from "./ProfileContext.jsx";
import Profile from "./Profile.jsx";
import ProfileAddress from "./ProfileAddress.jsx";
import {useState} from "react";
import ProfileForm from "./ProfileForm.jsx";

export default function ProfileApp() {
    const [name, setName] = useState("Dzaru");

    return (
        <>
            <ProfileContext.Provider value={name}>
                <h1>Profile App</h1>
                <ProfileForm name={name} setName={setName}/>
                <Profile/>
                <ProfileAddress/>
            </ProfileContext.Provider>
        </>
    )
}
```

**`src/profile/ProfileForm.jsx`**

```jsx
export default function ProfileForm({name, setName}) {
    function handleChange(e) {
        setName(e.target.value);
    }

    return (
        <>
            <h2>Profile Form</h2>
            <input type="text" value={name} onChange={handleChange} />
        </>
    )
}
```

### Reading Context with useContext

Any descendant reads the current value with `useContext()` — no Props needed.

> Reference: [react.dev/reference/react/useContext](https://react.dev/reference/react/useContext)

**`src/profile/Profile.jsx`**

```jsx
import {useContext} from "react";
import {ProfileContext} from "./ProfileContext.jsx";

export default function Profile() {
    const profile = useContext(ProfileContext);

    return (
        <>
            <h2>Profile Address</h2>
            <p>Hello {profile}</p>
        </>
    )
}
```

**`src/profile/ProfileAddress.jsx`**

```jsx
import {useContext} from "react";
import {ProfileContext} from "./ProfileContext.jsx";

export default function ProfileAddress() {
    const profile = useContext(ProfileContext);

    return (
        <>
            <h2>Profile Address</h2>
            <p>Address: {profile}</p>
        </>
    )
}
```

### Combining Context with a Reducer

A reducer's `dispatch` function can be shared through Context too — so a deeply nested Component can dispatch actions directly, without every Component in between having to forward an `onChange`/`onDelete` handler through Props. The Note app uses **two** contexts: one for the notes array, one for `dispatch`.

**`src/note/NoteContext.jsx`**

```jsx
import {createContext} from "react";

export const NotesContext = createContext(null)
export const NoteDispatchContext = createContext(null)
```

**`src/note/NoteApp.jsx`**

```jsx
import {useReducer} from "react";
import NoteForm from "./NoteForm.jsx";
import NoteList from "./NoteList.jsx";
import {useImmerReducer} from "use-immer";
import {NoteDispatchContext, NotesContext} from "./NoteContext.jsx";

let id = 0
const initialNotes = [
    {id: id++, text: "Learn HTML", done: false},
    {id: id++, text: "Learn CSS", done: false},
    {id: id++, text: "Learn JS", done: false},
]

export default function NoteApp() {
    const [notes, dispatch] = useImmerReducer(notesReducer, initialNotes);

    return (
        <div>
            <NotesContext.Provider value={notes}>
                <NoteDispatchContext.Provider value={dispatch}>
                    <h1>Note App</h1>
                    <NoteForm/>
                    <NoteList/>
                </NoteDispatchContext.Provider>
            </NotesContext.Provider>
        </div>
    )
}
```

`NoteForm` and `Note` then read `NoteDispatchContext` directly instead of receiving `onAddNote`/`onChange`/`onDelete` as Props:

**`src/note/NoteForm.jsx`**

```jsx
import {useContext, useState} from "react";
import {NoteDispatchContext} from "./NoteContext.jsx";

export default function NoteForm() {
    const [text, setText] = useState("")
    const dispatch = useContext(NoteDispatchContext)

    function handleChange(e) {
        setText(e.target.value)
    }

    function handleClick(e) {
        setText("")
        dispatch({
            type: "ADD_NOTE",
            text: text,
        })
    }

    return (
        <>
            <input placeholder="Add Note" value={text} onChange={handleChange}/>
            <button onClick={handleClick}>Add Note</button>
        </>
    )
}
```

**`src/note/Note.jsx`**

```jsx
import {useContext, useState} from "react";
import {NoteDispatchContext} from "./NoteContext.jsx";

export default function Note({note}) {
    const [isEditing, setEditing] = useState(false);
    const dispatch = useContext(NoteDispatchContext);

    let component;

    function handleChangeText(e) {
        dispatch({
            ...note,
            type: "CHANGE_NOTE",
            text: e.target.value,
        })
    }

    if (isEditing) {
        component = (
            <>
                <input value={note.text} onChange={handleChangeText}/>
                <button onClick={() => setEditing(false)}>Save</button>
            </>
        )
    } else {
        component = (
            <>
                {note.text}
                <button onClick={() => setEditing(true)}>Edit</button>
            </>
        )
    }

    function handleChangeDone(e) {
        dispatch({
            ...note,
            type: "CHANGE_NOTE",
            done: e.target.checked,
        })
    }

    function handleDelete(e) {
        dispatch({
            ...note,
            type: "DELETE_NOTE",
            id: note.id
        })
    }

    return (
        <label>
            <input type="checkbox" checked={note.done} onChange={handleChangeDone}/>
            {component}
            <button onClick={handleDelete}>Delete</button>
        </label>
    )
}
```

`NoteList` reads the shared `notes` array the same way, via `useContext(NotesContext)` — its full code (combined with `useMemo`) is in the [useMemo](#-usememo) section below.

---

## 📌 useRef

Sometimes a Component needs to remember something *without* triggering a re-render. That's what **Ref** Hooks are for, via `useRef()`.

> Reference: [react.dev/reference/react/useRef](https://react.dev/reference/react/useRef)

### Ref vs. State

| | `useRef(initial)` | `useState(initial)` |
| --- | --- | --- |
| Returns | `{ current: initial }` | `[value, setValue]` |
| Re-renders on change? | No | Yes |
| Mutable? | Yes — `current` can be reassigned directly | No — only via the setter |
| Safe to read/write during render? | No — read/write in an event handler instead | Yes, but each render sees its own snapshot |

### A Timer Built with useRef

`useRef()` returns an object with a `current` property, which holds whatever value the ref is pointing at — here, the interval ID from `setInterval`, which doesn't need to trigger a re-render itself.

**`src/timer/Timer.jsx`**

```jsx
import {useRef, useState} from "react";

export default function Timer() {
    const [start, setStart] = useState(null)
    const [now, setNow] = useState(null)

    const timer = useRef(null)

    function handleStart() {
        setStart(Date.now())
        setNow(Date.now())

        timer.current = setInterval(() => {
            setNow(Date.now())
        }, 10)
    }

    function handleStop() {
        clearInterval(timer.current)
    }

    return (
        <>
            <h1>Timer: {now - start} ms</h1>
            <button onClick={handleStart}>Start</button>
            <button onClick={handleStop}>Stop</button>
        </>
    );
}
```

### Manipulating the DOM with ref

React updates the DOM automatically on every render, so manual DOM access is rarely needed — but sometimes you do need it directly, e.g. moving keyboard focus to a specific element. Attach a `ref` attribute to the element to get a handle on the underlying DOM node.

> **Key Insight:** With React 19, a function component can accept `ref` as a plain prop — no `forwardRef` wrapper required, as seen in `GuestBookForm` below.

**`src/guestbook/GuestBookForm.jsx`**

```jsx
export default function GuestBookForm({ref, name, setName}) {
    return (
        <>
            <label htmlFor="name">Name</label> <br/>
            <input
                ref={ref}
                type="text"
                name="name"
                value={name} onChange={(e) => setName(e.target.value)}
            />
        </>
    )
}
```

**`src/guestbook/GuestBook.jsx`**

```jsx
import {useRef, useState} from "react";
import GuestBookForm from "./GuestBookForm.jsx";

export default function GuestBook() {
    const [name, setName] = useState('');
    const [message, setMessage] = useState('');

    const nameInput = useRef(null);

    function handleSubmit(e) {
        e.preventDefault();
        setName("");
        setMessage("");

        nameInput.current.focus();

        alert(`Name: ${name}, Message: ${message}`);
    }

    return (
        <>
            <h1>Guest Book</h1>
            <form>
                <GuestBookForm ref={nameInput} name={name} setName={setName}/>
                <div>
                    <label htmlFor="message">Message</label> <br/>
                    <textarea
                        name="message"
                        value={message}
                        onChange={(e) => setMessage(e.target.value)}
                    />
                </div>
                <button type="submit" onClick={handleSubmit}>Submit</button>
            </form>
        </>
    )
}
```

---

## 🔌 useEffect

Some Components need to talk to an **external system** — a non-React widget, the network, and so on. `useEffect()` lets you run code *after* render finishes, so a Component can synchronize with the outside world.

> Reference: [react.dev/reference/react/useEffect](https://react.dev/reference/react/useEffect)

### Why Event Handlers Aren't Enough

Components are expected to be pure, with side effects normally confined to event handlers. But sometimes a Component needs a side effect *just from rendering* — e.g. fetching data from a server as soon as it appears on screen. That's exactly the gap Effects fill.

### Fetching Data in an Effect

**`src/product/Product.jsx`**

```jsx
export default function Product({product}) {
    return (
        <div>
            <h2>{product.id} : {product.name}</h2>
            <p>Price: ${product.price}</p>
        </div>
    )
}
```

**`src/product/ProductList.jsx`**

```jsx
import {useEffect, useRef, useState} from "react";
import Product from "./Product.jsx";

export default function ProductList() {
    const [products, setProducts] = useState([])
    const [load, setLoad] = useState(false)

    function handleClick() {
        setLoad(true)
    }

    useEffect(() => {
        console.log("Call UseEffect with Empty Array")
    }, []);

    useEffect(() => {
        console.info("Call UseEffect")

        async function fetchProducts() {
            const response = await fetch("/products.json")
            const data = await response.json()
            setProducts(data)
        }

        if (load) {
            fetchProducts()
        }

        return () => {
            console.info("ProductList component unmounted");
        }
    }, [load])

    return (
        <>
            <h1>Product List</h1>
            <button onClick={handleClick}>Load Product</button>
            {products.map((product) => (
                <Product key={product.id} product={product}/>
            ))}
        </>
    )
}
```

### Effect Dependencies

By default, an Effect re-runs after *every* render. Passing a **dependency array** as the second argument tells React to skip the Effect unless one of those State/Prop values actually changed — the `[load]` array above means this Effect only re-runs when `load` changes.

Passing an **empty** array (`[]`) means the Effect runs exactly once, right after the first render, and never again — that's what the first `useEffect` above does.

### Async Code Inside an Effect

`useEffect`'s callback itself can't be `async` — but nothing stops you from declaring an `async` function *inside* the Effect and calling it, as `fetchProducts` does above.

### Effect Cleanup

An Effect can `return` a cleanup function, which React runs right before the Effect runs again, or when the Component is removed — similar to a `finally` block. See `useOnline` in [Custom Hooks](#-custom-hooks) below for a cleanup that removes event listeners it added.

### When Not to Use an Effect

- **Don't use Effects to initialize the app.** An Effect runs *after* render (and can re-run on every re-render) — one-time startup logic belongs in plain JavaScript outside React
- **Don't use Effects to write data to a server.** Because an Effect can re-run due to re-renders, it can trigger the same server write multiple times unintentionally — use an event handler instead, where you control exactly when it fires
- **In general, minimize Effects.** They're an escape hatch for synchronizing with something *outside* React — if there's no external system involved, you probably don't need one

---

## 🚀 useMemo

**Memoization** caches the result of an expensive calculation so it doesn't have to be redone on every render. `useMemo(callback, [dependencies])` is part of React's performance Hooks.

> Reference: [react.dev/reference/react/useMemo](https://react.dev/reference/react/useMemo)

### Memoizing an Expensive Calculation

The Note app's search box only needs to re-filter the notes list when either the notes or the search term actually changes — not on every render.

**`src/note/NoteList.jsx`**

```jsx
import Note from "./Note.jsx";
import {NotesContext} from "./NoteContext.jsx";
import {useContext, useMemo, useRef, useState} from "react";

export default function NoteList() {
    const notes = useContext(NotesContext)
    const [search, setSearch] = useState("");
    const searchInput = useRef(null);

    const filteredNotes = useMemo(() => {
        console.info('Filtering Notes');
        return notes.filter((note) => note.text.includes(search));
    }, [notes, search]);

    function handleSearch() {
        console.info("Search")
        setSearch(searchInput.current.value);
    }

    return (
        <>
            <input ref={searchInput} placeholder="Search"/>
            <button onClick={handleSearch}>Search</button>
            <ul>
                {filteredNotes.map(note => (
                    <li key={note.id}>
                        <Note note={note}/>
                    </li>
                ))}
            </ul>
        </>
    )
}
```

---

## 🪝 Custom Hooks

React ships with plenty of built-in Hooks, but you can also write your own. A custom Hook is just a function, named starting with `use`, that can call other Hooks internally.

### Extracting useOnline()

**`src/online/OnlineHook.jsx`**

```jsx
import {useEffect, useState} from "react";

export function useOnline() {
    const [isOnline, setIsOnline] = useState(true);

    useEffect(() => {
        function handleOnline() {
            setIsOnline(true)
        }

        function handleOffline() {
            setIsOnline(false)
        }

        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);

        return () => {
            window.removeEventListener('online', handleOnline)
            window.removeEventListener('offline', handleOffline)
        };
    })
    return isOnline;
}
```

**`src/online/Online.jsx`**

```jsx
import {useRef} from "react";
import {useOnline} from "./OnlineHook.jsx";

export default function Online() {
    const isOnline = useOnline();
    return (
        <h1>
            {isOnline ? 'Online' : 'Offline'}
        </h1>
    )
}
```

The listener cleanup (`removeEventListener`) is exactly why the Effect returns a cleanup function — without it, every re-render would attach another pair of listeners.

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Component** | Reusable function returning UI | `function Name() { return <jsx/> }` |
| **Props** | Data passed from parent to child | `function Name({ text }) { ... }` |
| **children** | Nested content passed to a Component | `function Container({ children })` |
| **Conditional Rendering** | Show different UI per condition | `return null`, `cond ? a : b` |
| **key** | Identify list items / reset State | `<Item key={id} />` |
| **Pure Component** | Same input → same output, no side effects | avoid mutating outside variables |
| **Event Handler** | Respond to user interaction | `onClick={handleClick}` |
| **useState** | Local, re-render-triggering memory | `const [v, setV] = useState(init)` |
| **useReducer** | Consolidated State-update logic | `const [s, dispatch] = useReducer(fn, init)` |
| **createContext / useContext** | Share a value across the tree | `useContext(MyContext)` |
| **useRef** | Persistent value, no re-render | `const ref = useRef(init)` |
| **useEffect** | Sync with an external system | `useEffect(fn, [deps])` |
| **useMemo** | Cache an expensive calculation | `useMemo(fn, [deps])` |
| **Custom Hook** | Reusable stateful logic | `function useThing() { ... }` |

---

## 💡 Best Practices

**✅ Do This**

- **Treat State as immutable** — always build a new object/array instead of mutating the existing one
- **Reach for Immer (`use-immer`)** once object/array updates get nested or repetitive — it reads far more naturally than manual spreading
- **Give list items a stable `key`** pulled from the data (like an `id`), not the array index, whenever the list can reorder
- **Use `key` deliberately to reset State** when two instances of the same Component at the same position need independent State
- **Model complex, multi-action State with a reducer** — describe *what happened* via `dispatch`, and let the reducer decide *what changes*
- **Reach for Context** once the same value needs to reach many Components, instead of drilling Props through several layers
- **Add a cleanup function to an Effect** whenever it subscribes to something (event listeners, intervals, sockets)

**❌ Avoid This**

- **Mutating State directly** (`state.field = x`, `array.push(...)`) — React can't detect the mutation, so nothing re-renders
- **Writing an impure Component** — reading or mutating variables outside the Component's own scope makes its output unpredictable
- **Forgetting the dependency array on `useEffect`** — an Effect with no array re-runs after every render, which can spiral into an infinite loop if it also updates State
- **Using an Effect where an event handler would do** — Effects are for synchronizing with something *outside* React, not for reacting to user actions
- **Reading or writing a `ref`'s `current` value during render** — that's undefined behavior; only touch it in event handlers or Effects
- **Calling `setState` and expecting the current render's variable to reflect it immediately** — State updates apply to the *next* render's snapshot, not the current one

> Reference: [react.dev/reference/react](https://react.dev/reference/react)
