# Study React ⚛️

This repository contains a comprehensive reference guide for React — covering the fundamentals (JSX, props, state, hooks, context), client-side routing with React Router, global state management with Redux, and a complete Contact Management SPA built end-to-end against an existing REST API.

## List of Material 📚

- ⚛️ **[React Basics](001-react-basics.md)**

  JSX, props, state, hooks, and context, built hands-on across a multi-page Vite playground — from a first component to `useEffect`, `useReducer`, and custom hooks:

  ```jsx
  function HeaderHelloWorld({ text = "Ups, I forgot include the text" }) {
    return (
      <h1 className="title">
        {text.toUpperCase()}
      </h1>
    )
  }
  ```

- 🧭 **[React Router](002-react-router.md)**

  Client-side routing fundamentals — `BrowserRouter`, nested routes, index routes, layouts with `<Outlet>`, star segments, and programmatic navigation:

  ```jsx
  import { BrowserRouter, Route, Routes } from "react-router";

  createRoot(document.getElementById('root')).render(
    <StrictMode>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </BrowserRouter>
    </StrictMode>,
  )
  ```

- 🗃️ **[ReactJS Redux](003-react-redux.md)**

  Global state management with Redux Toolkit and `react-redux` — the Store, Slices, Actions, Selectors, and Immer's role in keeping "mutations" safe:

  ```js
  import { createSlice } from "@reduxjs/toolkit";

  export const counterSlice = createSlice({
    name: "counter",
    initialState: 0,
    reducers: {}
  })
  ```

- 🚀 **[React.js Study Case — Contact Management](004-react-study-case.md)**

  A full SPA built stage by stage across three modules — User → Contact → Address — consuming an existing Node.js/Express/Prisma REST API, plus a full debugging journal of every real integration bug hit along the way:

  ```javascript
  export const userDetail = async (token) => {
    return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        Authorization: token,
      },
    });
  };
  ```

## 📍 References

- [Udemy](https://www.udemy.com/course/belajar-reactjs/?srsltid=AfmBOoqNAcyps0sY43CqZg0PBLkgREw33ohK1zieXbgDAaXBotOmz_ZQ&couponCode=CP260817G2)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
