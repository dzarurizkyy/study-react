# 🗃️ ReactJS Redux

A practical reference guide for learning Redux with React — covering the Store, Slices, Actions, Selectors, and wiring it all up with `react-redux` and `react-router`, built hands-on inside the `belajar-reactjs-redux` Vite playground.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [Why Global State Management?](#why-global-state-management)
  - [What Is Redux?](#what-is-redux)
- [Project Setup](#-project-setup)
  - [Creating the Project](#creating-the-project)
  - [Adding Redux](#adding-redux)
- [Provider & Store](#-provider--store)
  - [Wrapping the App in a Provider](#wrapping-the-app-in-a-provider)
  - [Creating the Store with configureStore](#creating-the-store-with-configurestore)
- [State with Slices](#-state-with-slices)
  - [Creating a Slice](#creating-a-slice)
  - [Registering the Reducer](#registering-the-reducer)
  - [Reading State with useSelector](#reading-state-with-useselector)
  - [Why There's No Setter: Immutable State](#why-theres-no-setter-immutable-state)
- [Actions](#-actions)
  - [Reducers Are Actions](#reducers-are-actions)
  - [Immer Under the Hood](#immer-under-the-hood)
  - [Dispatching Actions with useDispatch](#dispatching-actions-with-usedispatch)
  - [Action Payloads](#action-payloads)
- [Global State in Practice](#-global-state-in-practice)
- [Selectors](#-selectors)
  - [Custom Selectors](#custom-selectors)
  - [Selectors with Parameters](#selectors-with-parameters)
- [TodoList: CRUD Across Multiple Pages](#-todolist-crud-across-multiple-pages)
  - [TodoList State & Actions](#todolist-state--actions)
  - [Adding a Todo](#adding-a-todo)
  - [Updating a Todo](#updating-a-todo)
  - [Listing & Deleting Todos](#listing--deleting-todos)
  - [Wiring Up the Routes](#wiring-up-the-routes)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### Why Global State Management?

- Previously, State Management in React was covered — how to manage data *inside* a single component
- Context was also covered — how to pass data from a parent component down to its children
- But what if some data needs to be accessed truly **globally** — from any component, anywhere, even across completely unrelated component trees? That's Global State Management
- The answer is a dedicated place to hold that State, commonly called a **Store**

### What Is Redux?

- **Redux** is a Store library for holding State
- It lets State be stored once and accessed from any component or page
- This makes managing State across many components far easier than wiring Context through every layer
- Redux is a popular, open-source project for Global State Management

> Reference: [redux.js.org](https://redux.js.org/)

---

## 🏗️ Project Setup

### Creating the Project

```bash
npm create vite@latest belajar-reactjs-redux -- --template react
```

React was then updated to version 19, and `react-router` was added for multi-page navigation:

```bash
npm install react-router
```

### Adding Redux

Redux itself is split across two packages: the core toolkit, and the React bindings.

```bash
npm install @reduxjs/toolkit
npm install react-redux
```

---

## 🔌 Provider & Store

### Wrapping the App in a Provider

Before Redux can be used, the app needs to be wrapped in the `Provider` component that ships with Redux. `Provider` takes one required prop — `store` — which is covered next.

### Creating the Store with configureStore

A **Store** is where State data actually lives. Before using any State, a Store has to be created first — later, State is registered into that Store. The Store is built with the `configureStore()` function.

> Reference: [redux-toolkit.js.org/api/configureStore](https://redux-toolkit.js.org/api/configureStore)

**`belajar-reactjs-redux/src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'
import { Provider } from 'react-redux'
import { BrowserRouter, Route, Routes } from 'react-router'
import { configureStore } from '@reduxjs/toolkit'

const store = configureStore({
  reducer: {}
})

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <Provider store={store}>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<App />} />
        </Routes>
      </BrowserRouter>
    </Provider>
  </StrictMode>,
)
```

---

## 🧠 State with Slices

### Creating a Slice

Once the Store exists, State can be added to it. The integration between Store and State happens through a **Reducer** — and Redux bundles State and its Reducer together into a single concept called a **Slice**, built with `createSlice()`. When creating State, an initial value must always be given.

> Reference: [redux-toolkit.js.org/api/createSlice](https://redux-toolkit.js.org/api/createSlice)

**Task:** build a Slice for a Counter feature, in a file named `counterSlice.js`.

**`belajar-reactjs-redux/src/counterSlice.js`**

```js
import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter",
  initialState: 0,
  reducers: {}
})
```

### Registering the Reducer

After the Slice is created, its Reducer has to be registered into the Store's `reducer` map — that's what actually connects the two.

**`belajar-reactjs-redux/src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'
import { Provider } from 'react-redux'
import { BrowserRouter, Route, Routes } from 'react-router'
import { configureStore } from '@reduxjs/toolkit'
import { counterSlice } from './counterSlice'
import Counter from './Counter'

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
})

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <Provider store={store}>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<App />} />
          <Route path="/counter" element={<Counter />}/>
        </Routes>
      </BrowserRouter>
    </Provider>
  </StrictMode>,
)
```

### Reading State with useSelector

To read State from the Store, use the `useSelector()` function — pass it a callback that picks out whichever piece of State is needed, matching the name it was registered under in the Store.

> Reference: [react-redux.js.org/api/hooks#useselector](https://react-redux.js.org/api/hooks#useselector)

**`belajar-reactjs-redux/src/Counter.jsx`**

```jsx
import { useSelector } from "react-redux";

export default function Counter() {
  const counter = useSelector(state => state.counter)

  return (
    <div>
      <h1>Counter: {counter}</h1>
    </div>
  )
}
```

### Why There's No Setter: Immutable State

With component-local State, React hands back a setter function to change the value. Redux gives no such thing — there's no function to directly mutate State. Changing data in a Redux Store always goes through an **Action** instead.

---

## ⚡ Actions

### Reducers Are Actions

What's called an Action in Redux is really just a Reducer, made easier to work with. Writing a Reducer function inside a Slice's `reducers` automatically exposes its name as a dispatchable Action.

**Task:** add two Reducers to `counterSlice` — `increment(state)` and `decrement(state)`.

### Immer Under the Hood

React encourages treating State as immutable. When using plain `useState`, that meant reaching for a library like Immer to make immutable updates less tedious. Redux uses Immer **automatically** under the hood whenever a Reducer runs — so State can be "mutated" directly inside a Reducer without manually copying data; Immer handles the copy for you.

### Dispatching Actions with useDispatch

Once Actions exist as Reducers, they're triggered the same way any Reducer would be — via `dispatch`. Create a dispatch function with the `useDispatch()` hook, then call it with the Action that identifies which Reducer to run.

> Reference: [react-redux.js.org/api/hooks#usedispatch](https://react-redux.js.org/api/hooks#usedispatch)

### Action Payloads

An Action can also carry a parameter. Whatever value is passed into an Action is automatically placed on a `payload` attribute of the second argument to the Reducer. For example, `dispatch(increment(2))` results in the Reducer receiving `(state, action)`, where `action.type` is `"counter/increment"` and `action.payload` is `2`. The payload can hold any data type — a plain number or string, or something more complex like an array or object.

**Task:** extend `increment` / `decrement` so they accept an optional value to increment or decrement by.

**`belajar-reactjs-redux/src/counterSlice.js`**

```js
import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter",
  initialState: 0,
  reducers: {
    increment: (state, action) => {
      if (action.payload) {
        return state + action.payload
      }
      return state + 1
    },
    decrement: (state, action) => {
      if (action.payload) {
        return state - action.payload
      }
      return state - 1
    }
  }
})

export const { increment, decrement } = counterSlice.actions
```

**`belajar-reactjs-redux/src/Counter.jsx`**

```jsx
import { useDispatch, useSelector } from "react-redux";
import { decrement, increment } from "./counterSlice";

export default function Counter() {
  const counter = useSelector(state => state.counter)
  const dispatch = useDispatch()

  function handleIncrement() {
    dispatch(increment())
  }

  function handleDecrement() {
    dispatch(decrement())
  }

  function handleIncrementBy(value) {
    dispatch(increment(value))
  }

  function handleDecrementBy(value) {
    dispatch(decrement(value))
  }

  return (
    <div>
      <h1>Counter: {counter}</h1>
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={handleDecrement}>Decrement</button>
      <button onClick={() => handleIncrementBy(10)}>Increment By 10</button>
      <button onClick={() => handleDecrementBy(10)}>Decrement By 10</button>
    </div>
  )
}
```

---

## 🌍 Global State in Practice

State kept in a Store is genuinely accessible from any component. To prove this, the same `Counter` component is rendered twice at `/counter` — both instances read and update the exact same `counter` value in the Store, unlike component-local `useState`, where every instance would keep its own separate value.

**`belajar-reactjs-redux/src/main.jsx`** (excerpt)

```jsx
<Route path="/counter" element={
  <>
    <Counter />
    <Counter />
  </>
}/>
```

---

## 🔍 Selectors

### Custom Selectors

`useSelector` is enough to read raw State, but sometimes a more derived value is needed — and a Reducer/Action isn't the right tool for that, since Reducers exist to *change* State, not read it. For that, a Slice can define its own **Selectors**, built the same way as Reducers/Actions.

**Task:** add a `getDoubleCounter()` selector that returns the counter value multiplied by 2.

### Selectors with Parameters

Like Actions, a Selector can also take a parameter — for example, a selector that lets the caller choose the multiplier.

**`belajar-reactjs-redux/src/counterSlice.js`**

```js
import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter",
  initialState: 0,
  reducers: {
    increment: (state, action) => {
      if (action.payload) {
        return state + action.payload
      }
      return state + 1
    },
    decrement: (state, action) => {
      if (action.payload) {
        return state - action.payload
      }
      return state - 1
    }
  },
  selectors: {
    getDoubleCounter(state) {
      return state * 2
    },
    getCounter(state, value) {
      return state * value
    }
  }
})

export const { increment, decrement } = counterSlice.actions
export const { getDoubleCounter, getCounter } = counterSlice.selectors
```

**`belajar-reactjs-redux/src/Counter.jsx`**

```jsx
import { useDispatch, useSelector } from "react-redux";
import { decrement, getCounter, getDoubleCounter, increment } from "./counterSlice";

export default function Counter() {
  const counter = useSelector(state => state.counter)
  const doubleCounter = useSelector(getDoubleCounter)
  const tripleCounter = useSelector(state => getCounter(state, 3))

  const dispatch = useDispatch()

  function handleIncrement() {
    dispatch(increment())
  }

  function handleDecrement() {
    dispatch(decrement())
  }

  function handleIncrementBy(value) {
    dispatch(increment(value))
  }

  function handleDecrementBy(value) {
    dispatch(decrement(value))
  }

  return (
    <div>
      <h1>Counter: {counter}</h1>
      <h1>Double Counter: {doubleCounter}</h1>
      <h1>Triple Counter: {tripleCounter}</h1>
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={handleDecrement}>Decrement</button>
      <button onClick={() => handleIncrementBy(10)}>Increment By 10</button>
      <button onClick={() => handleDecrementBy(10)}>Decrement By 10</button>
    </div>
  )
}
```

---

## ✅ TodoList: CRUD Across Multiple Pages

**Task:** build a TodoList feature that spans multiple pages — pages managed with React Router, data managed with Redux.

### TodoList State & Actions

The first step is a `todoList` State in the Store — an array of Todo objects, each with an `id` and a `name`, defined in `src/todoListSlice.js`. Three Actions are added on top of it:

- `addTodo({ name })` — add a todo
- `removeTodo({ id })` — remove a todo
- `updateTodo({ id, name })` — update a todo

A `getTodo` selector is also added, to look up a single todo by `id`.

**`belajar-reactjs-redux/src/todoListSlice.js`**

```js
import { createSlice } from "@reduxjs/toolkit";

let id = 0;
export const todoListSlice = createSlice({
  name: "todoList",
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      const { name } = action.payload;
      state.push({ name, id: id++ });
    },
    removeTodo: (state, action) => {
      const { id } = action.payload;
      const index = state.findIndex((todo) => todo.id === id);
      if (index !== -1) {
        state.splice(index, 1);
      }
    },
    updateTodo: (state, action) => {
      const { id, name } = action.payload;
      const todo = state.find((todo) => todo.id === id);
      if (todo) todo.name = name;
    },
  },
  selectors: {
    getTodo: (state, id) => state.find((todo) => todo.id === id),
  },
});

export const { addTodo, removeTodo, updateTodo } = todoListSlice.actions;
export const { getTodo } = todoListSlice.selectors;
```

### Adding a Todo

`AddTodo` keeps the new name in local `useState`, then dispatches `addTodo` and navigates back to the list with `useNavigate()` once it's saved.

**`belajar-reactjs-redux/src/AddTodo.jsx`**

```jsx
import { useState } from "react"
import { useDispatch } from "react-redux"
import { useNavigate } from "react-router"
import { addTodo } from "./todoListSlice"

export default function AddTodo() {
  const [name, setName] = useState("")
  const dispatch = useDispatch()
  const navigate = useNavigate()

  const handleClick = () => {
    dispatch(addTodo({ name }))
    navigate("/todolist")
  }

  return (
    <div>
      <h1>Add Todo</h1>
      <input type={"text"} placeholder={"Enter todo name"} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleClick}>Add</button>
    </div>
  )
}
```

### Updating a Todo

`UpdateTodo` reads the `id` from the route params via `useParams()`, looks the todo up with the `getTodo` selector, and dispatches `updateTodo` on save.

**`belajar-reactjs-redux/src/UpdateTodo.jsx`**

```jsx
import { useState } from "react"
import { useDispatch, useSelector } from "react-redux"
import { useNavigate, useParams } from "react-router"
import { getTodo, updateTodo } from "./todoListSlice"

export default function UpdateTodo() {
  const params = useParams()
  const todo = useSelector((state) => getTodo(state, Number(params.id)))

  const [name, setName] = useState(todo.name)
  const dispatch = useDispatch()
  const navigate = useNavigate()

  const handleClick = () => {
    dispatch(updateTodo({ id: todo.id, name }))
    navigate("/todolist")
  }

  return (
    <div>
      <h1>Update Todo</h1>
      <input type={"text"} value={name} placeholder={"Enter todo name"} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleClick}>Update</button>
    </div>
  )
}
```

### Listing & Deleting Todos

`ListTodo` reads the whole `todoList` array with `useSelector`, links out to the Add/Update pages with `react-router`'s `Link`, and dispatches `removeTodo` directly from the row's Delete button.

**`belajar-reactjs-redux/src/ListTodo.jsx`**

```jsx
import { useDispatch, useSelector } from "react-redux";
import { Link } from "react-router";
import { removeTodo } from "./todoListSlice"

export default function ListTodo() {
  const todos = useSelector((state) => state.todoList)
  const dispatch = useDispatch()

  function handleDeleteTodo(id) {
    dispatch(removeTodo({ id }))
  }

  return (
    <div>
      <h1>List Todo</h1>
      <Link to={"/todolist/add"}>
        Add Todo
      </Link>
      <table>
        <thead>
          <th>Id</th>
          <th>Name</th>
          <th>Action</th>
        </thead>
        <tbody>
          {todos.map((todo) => (
            <tr key={todo.id}>
              <td>{todo.id}</td>
              <td>{todo.name}</td>
              <td>
                <Link to={`/todolist/${todo.id}/edit`}>
                  Update
                </Link>
                <button onClick={() => handleDeleteTodo(todo.id)}>
                  Delete
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}
```

### Wiring Up the Routes

Both Slices are registered in the Store, and each TodoList page gets its own route.

**`belajar-reactjs-redux/src/main.jsx`**

```jsx
import './index.css'
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'
import { Provider } from 'react-redux'
import { BrowserRouter, Route, Routes } from 'react-router'
import { configureStore } from '@reduxjs/toolkit'
import { counterSlice } from './counterSlice'
import Counter from './Counter'
import { todoListSlice } from './todoListSlice'
import TodoList from './ListTodo'
import UpdateTodo from './UpdateTodo'
import AddTodo from './AddTodo'

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
    todoList: todoListSlice.reducer
  }
})

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <Provider store={store}>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<App />} />
          <Route path="/counter" element={
            <>
              <Counter />
              <Counter />
            </>
          }/>
          <Route path='/todolist' element={<TodoList />} />
          <Route path='/todolist/add' element={<AddTodo />} />
          <Route path='/todolist/:id/edit' element={<UpdateTodo />} />
        </Routes>
      </BrowserRouter>
    </Provider>
  </StrictMode>,
)
```

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Store** | Central place where global State lives | `configureStore({ reducer: {...} })` |
| **Provider** | Makes the Store available to the component tree | `<Provider store={store}>` |
| **Slice** | Bundle of State + its Reducer for one feature | `createSlice({ name, initialState, reducers })` |
| **Reducer / Action** | Function describing how State changes | defined under `reducers`, called via `dispatch(actionName())` |
| **useSelector** | Read State from the Store | `useSelector(state => state.counter)` |
| **useDispatch** | Get a `dispatch` function to trigger Actions | `const dispatch = useDispatch()` |
| **Payload** | Data sent along with an Action | `dispatch(increment(2))` → `action.payload === 2` |
| **Selectors** | Slice-defined derived reads of State | `selectors: { getDoubleCounter(state) {...} }` |
| **Immer** | Library Redux uses automatically for immutable updates | mutate `state` directly inside a Reducer |

---

## 💡 Best Practices

**✅ Do This**

- **Group related State and its Reducer into one Slice** with `createSlice()`, instead of scattering update logic across components
- **Register every Slice's reducer in the Store** so its State and Actions are actually wired up and usable
- **Let Reducers "mutate" `state` directly** — Redux Toolkit runs Immer under the hood, so this stays immutable under the surface
- **Use Selectors for derived reads** (e.g. `getDoubleCounter`) instead of duplicating that computation inside every component
- **Reach for a Store when State genuinely needs to be global** — shared across unrelated components, not just parent/child

**❌ Avoid This**

- **Expecting a setter function from Redux** — there isn't one; all State changes go through `dispatch` + an Action
- **Writing an Action to *read* State** — that's what Selectors are for; Reducers exist to change State, not fetch it
- **Manually copying State before mutating it inside a Reducer** — Immer already does that automatically
- **Forgetting to register a new Slice's reducer in `configureStore`** — the State exists on the Slice but won't actually be reachable from the Store

> Reference: [redux.js.org](https://redux.js.org/) · [redux-toolkit.js.org](https://redux-toolkit.js.org/) · [react-redux.js.org](https://react-redux.js.org/)
