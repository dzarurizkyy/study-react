# 🧭 React Router

A practical reference guide for learning client-side routing in React — covering SPA fundamentals, nested routes, layouts, navigation, and URL-driven state, built hands-on inside this Vite playground.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [Client-Side vs Server-Side Rendering](#client-side-vs-server-side-rendering)
  - [Multi Page Application](#multi-page-application)
  - [Single Page Application](#single-page-application)
  - [Router Library](#router-library)
  - [Why React Router](#why-react-router)
- [Project Setup](#-project-setup)
  - [Creating the Project](#creating-the-project)
  - [Adding React Router](#adding-react-router)
  - [BrowserRouter](#browserrouter)
- [Routing Basics](#-routing-basics)
  - [Routes & Route](#routes--route)
- [Nested Routes](#-nested-routes)
  - [Index Route](#index-route)
- [Layouts with Outlet](#-layouts-with-outlet)
- [Star Segments](#-star-segments)
  - [Star Segment as a Not Found Page](#star-segment-as-a-not-found-page)
- [Navigation](#-navigation)
  - [Link Element](#link-element)
  - [Link `to` as an Object](#link-to-as-an-object)
  - [NavLink](#navlink)
- [Programmatic Navigation](#-programmatic-navigation)
  - [useNavigate](#usenavigate)
  - [useSearchParams](#usesearchparams)
  - [useLocation](#uselocation)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### Client-Side vs Server-Side Rendering

When building a web app, there are generally two rendering approaches: **CSR** (Client Side Rendering) and **SSR** (Server Side Rendering). When using a frontend framework like React, the default choice is usually CSR — the server no longer needs to render HTML pages, since everything is handled on the client by React. CSR is typically faster than SSR because communication with the server can be kept minimal, and the amount of data sent back and forth is smaller.

### Multi Page Application

When first learning React basics, it's common to create many separate HTML pages, where each page renders a React component. Even though this setup uses CSR, it's still technically a **Multi Page Application (MPA)**. The problem with an MPA is that every time the page changes, the browser has to reload and restart the entire React program from scratch. This means that even with CSR, page transitions aren't fully optimized, since everything gets reloaded on every navigation.

### Single Page Application

The opposite of an MPA is a **Single Page Application (SPA)**, where the app only ever loads a single HTML page. That page handles every request, and its content changes dynamically based on what the user asks for or what the server responds with. The benefit of an SPA is faster page transitions, since only the content changes — there's no need to reload the entire page. Unfortunately, React doesn't support SPA behavior well out of the box; doing it manually is complex and error-prone.

### Router Library

Instead of manually swapping page content, we can use a dedicated **Router Library** — a library that displays different components depending on the URL being accessed. This lets us focus on building components for each page without worrying about the complexity of determining which component to render for a given URL. Fortunately, there are many router libraries available for React.

> Reference: [react.libhunt.com/libs/router/react](https://react.libhunt.com/libs/router/react)

### Why React Router

There are many router libraries available for React, but in this material we'll use one of the most popular ones: **React Router** — an open-source routing library that is currently very popular in the React ecosystem.

> Reference: [reactrouter.com](https://reactrouter.com/) · [github.com/remix-run/react-router](https://github.com/remix-run/react-router)

---

## 🚀 Project Setup

### Creating the Project

```bash
npm create vite@latest belajar-reactjs-router -- --template react
```

Then upgrade React to version 19.

### Adding React Router

```bash
npm install react-router
```

### BrowserRouter

Unlike an MPA, when building an SPA we only need a single HTML page as the entry point for every request. React Router then takes care of deciding which component should be displayed. To enable this, wrap the root of the React app with the `BrowserRouter` component.

> Reference: [react_router.BrowserRouter](https://api.reactrouter.com/v7/functions/react_router.BrowserRouter.html)

**`src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'
import { BrowserRouter } from "react-router";

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
)
```

---

## 🔀 Routing Basics

### Routes & Route

**Routing** is the mapping between a URL path and the component that should be displayed for it. To define routes, React Router provides the `<Routes>` component, and for each individual mapping, the `<Route>` component.

> Reference: [react_router.Routes](https://api.reactrouter.com/v7/functions/react_router.Routes.html) · [react_router.Route](https://api.reactrouter.com/v7/functions/react_router.Route.html)

**Task:** add two components, `Home` and `About`, and map path `/` to `Home` and `/about` to `About`.

---

## 🪆 Nested Routes

Route definitions often share a common prefix. Writing each one out individually would be inefficient, so React Router supports **Nested Routes** — a `<Route>` can be declared inside another `<Route>`, and the parent's path is automatically used as a prefix for its children.

**Task:** create `Product`, `Customer`, and `Seller` components, then define three nested routes: `/data/products` → `Product`, `/data/customers` → `Customer`, and `/data/sellers` → `Seller`.

### Index Route

The `<Route>` component has a special `index` attribute, used to define the default page for a path — typically for the root path `/`, or for a nested path like `/data`. Without an index route, visiting `/data` directly would render nothing. Adding an `index` route lets us render a chosen component for that base path — in this case, a `Data` component.

**`src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { BrowserRouter, Route, Routes } from "react-router";
import Home from "./Home.jsx";
import About from "./About.jsx";
import Product from "./Product.jsx";
import Customer from "./Customer.jsx";
import Seller from "./Seller.jsx";
import Data from "./Data.jsx";

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/data">
          <Route index element={<Data />} />
          <Route path="products" element={<Product />} />
          <Route path="customers" element={<Customer />} />
          <Route path="sellers" element={<Seller />} />
        </Route>
      </Routes>
    </BrowserRouter>
  </StrictMode>
)
```

---

## 🪟 Layouts with Outlet

Several pages in a web app often share the same layout — the same header and footer, but different content in between. This could be done with plain JSX using `children`, but React Router offers an easier way: create a `Route` whose element is used as a **Layout**, then nest child routes inside it. Instead of using the `children` prop, the layout renders an `<Outlet>` component where the matched child route should appear.

> Reference: [react_router.Outlet](https://api.reactrouter.com/v7/functions/react_router.Outlet.html)

**Task:** create a `DataLayout` component and use it as the shared layout for every Data-related page.

**`src/DataLayout.jsx`**

```jsx
import { Outlet } from "react-router";

export default function DataLayout() {
  return (
    <>
      <div>
        <h1>This is header</h1>
      </div>
      <div>
        <Outlet />
      </div>
      <div>
        <h1>This is Footer</h1>
      </div>
    </>
  )
}
```

**`src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { BrowserRouter, Route, Routes } from "react-router";
import Home from "./Home.jsx";
import About from "./About.jsx";
import Product from "./Product.jsx";
import Customer from "./Customer.jsx";
import Seller from "./Seller.jsx";
import Data from "./Data.jsx";
import DataLayout from "./DataLayout.jsx";

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/data" element={<DataLayout />}>
          <Route index element={<Data />} />
          <Route path="products" element={<Product />} />
          <Route path="customers" element={<Customer />} />
          <Route path="sellers" element={<Seller />} />
        </Route>
      </Routes>
    </BrowserRouter>
  </StrictMode>
)
```

---

## ⭐ Star Segments

If a route path ends with `/*` (a **star segment**), it matches any characters, including the `/` character itself. For example, a path of `/files/*` matches both `/files/example` and `/files/more/example`. The value captured by the star segment can be read via `useParams()`.

**Task:** create an `Image` component mapped to path `/images/*`.

**`src/Image.jsx`**

```jsx
import { useParams } from "react-router"

export default function Image() {
    const params = useParams()
    return (
        <>
            <h1>Image</h1>
            <p>Image Page: {params["*"]}</p>
        </>
    )
}
```

### Star Segment as a Not Found Page

What happens when we visit a URL path that isn't registered in the router? We get an error saying no route matches the requested path. A common pattern in React Router is to add a catch-all star segment `/*` at the bottom of the route list, used as the handler for a **Not Found** page.

> **Key Insight:** Route matching order matters — the catch-all `path="*"` route must be declared last, otherwise it would swallow paths meant for more specific routes.

**Task:** create a `NotFound` component and register it under path `/*`.

**`src/NotFound.jsx`**

```jsx
import { useParams } from "react-router";

export default function NotFound() {
  const params = useParams()

  return (
    <>
      <h1>Not Found: {params['*']}</h1>
      <p>The page you looking for does not exist</p>
    </>
  )
}
```

**`src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { BrowserRouter, Route, Routes } from "react-router";
import Home from "./Home";
import Product from "./Product";
import Customer from "./Customer";
import Seller from "./Seller";
import Data from "./Data";
import DataLayout from "./DataLayout";
import About from './About';
import Image from './Image';
import NotFound from './NotFound';

createRoot(document.getElementById('root')).render(
    <StrictMode>
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home/>}/>
                <Route path="/about" element={<About />} />
                <Route path="/images/*" element={<Image />} />
                <Route path="/data" element={<DataLayout/>}>
                    <Route index element={<Data/>}/>
                    <Route path="products" element={<Product/>}/>
                    <Route path="customers" element={<Customer/>}/>
                    <Route path="sellers" element={<Seller/>}/>
                </Route>
                <Route path='*' element={<NotFound />}/>
            </Routes>
        </BrowserRouter>
    </StrictMode>
)
```

---

## 🔗 Navigation

One of the most common things we do in a web app is move from one page to another. Normally, page navigation uses an anchor element (`<a>`). The problem with anchor elements is that the browser performs a full page reload — meaning the entire page is reloaded, and every React object gets recreated from scratch. This can slow down page transitions considerably.

**Task:** update `DataLayout` so it has a menu at the top linking to every Data page.

### Link Element

React Router provides a `<Link>` component as a replacement for the anchor element. Unlike an anchor element, `<Link>` does not trigger a full page reload.

> Reference: [react_router.Link](https://api.reactrouter.com/v7/functions/react_router.Link.html)

### Link `to` as an Object

The `to` attribute of `<Link>` can also accept an object, with `pathname`, `search`, and `hash` attributes:

| Attribute | Purpose |
| --- | --- |
| `pathname` | the target path location |
| `search` | the query parameters |
| `hash` | the `#fragment` |

### NavLink

A common requirement when building links is to style the currently active link differently from inactive ones — usually by checking whether the current URL path matches the link's destination. React Router provides a dedicated component for this: `<NavLink>`. It's used just like `<Link>`, but when the current URL path matches its destination, it automatically applies the `a.active` style.

> Reference: [react_router.NavLink](https://api.reactrouter.com/v7/functions/react_router.NavLink.html)

**`src/DataLayout.jsx`**

```jsx
import { Link, NavLink, Outlet } from "react-router";
import "./data.css"

export default function DataLayout() {
    return (
        <>
            <div>
                <h1>This is header</h1>
            </div>
            <div>
                <ul>
                    {/* Reload Page */}
                    {/* <li><a href="/data/products">Products</a></li>
                    <li><a href="/data/customers">Customers</a></li>
                    <li><a href="/data/sellers">Sellers</a></li> */}
                    {/* There is no active link style */}
                    {/* <li><Link to={{
                        pathname: "products",
                        search: "?category=shoes",
                        hash: "#top"
                    }}>Products</Link></li>
                    <li><Link to="customers">Customers</Link></li>
                    <li><Link to="sellers">Sellers</Link></li> */}
                    {/* Add active link style */}
                    <li><NavLink to={{
                        pathname: "products",
                        search: "?category=shoes",
                        hash: "#top"
                    }}>Products</NavLink></li>
                    <li><NavLink to="customers">Customers</NavLink></li>
                    <li><NavLink to="sellers">Sellers</NavLink></li>
                </ul>
            </div>
            <div>
                <Outlet/>
            </div>
            <div>
                <h1>This is Footer</h1>
            </div>
        </>
    )
}
```

---

## 🧩 Programmatic Navigation

### useNavigate

Sometimes we need to navigate from one page to another using plain JavaScript. React Router provides the `useNavigate()` hook for this — it returns a function that can be called to navigate to another page, accepting either a `Path` object or a number (to move forward/backward in history).

> Reference: [react_router.useNavigate](https://api.reactrouter.com/v7/functions/react_router.useNavigate.html) · [NavigateFunction](https://api.reactrouter.com/v7/interfaces/react_router.NavigateFunction.html) · [Path](https://api.reactrouter.com/v7/interfaces/react_router.Path.html)

**Task:** add a button on the Home page that navigates to the Data page.

**`src/Home.jsx`**

```jsx
import { useNavigate } from "react-router"

export default function Home() {
    const navigate = useNavigate()

    function handleClick() {
        navigate('/data')
    }

    return (
        <>
            <h1>Home</h1>
            <p>Welcome to the home page!</p>
            <button onClick={handleClick}>Go to Data Page</button>
        </>
    )
}
```

### useSearchParams

URLs often carry data through query parameters. React Router provides the `useSearchParams()` hook to read them — it returns an array containing a `URLSearchParams` object and its setter function.

> Reference: [react_router.useSearchParams](https://api.reactrouter.com/v7/functions/react_router.useSearchParams.html) · [MDN: URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

**`src/ProductSearch.jsx`**

```jsx
import { useState } from "react"
import { useNavigate, useSearchParams } from "react-router"

export default function ProductSearch() {
  const [searchParams] = useSearchParams()
  const navigate = useNavigate()

  const [search, setSearch] = useState(searchParams.get('search') || "")

  function handleSearch() {
    navigate({
      pathname: "/data/products/search",
      search: `?search=${search}`
    })
  }

  return (
    <>
      <h1>Search Product</h1>
      <input type="text" value={search} onChange={(e) => setSearch(e.target.value)} />
      <button onClick={handleSearch}>Search</button>

      <p>
        You are searching for : {searchParams.get('search')}
      </p>
    </>
  )
}
```

**`src/main.jsx`**

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { BrowserRouter, Route, Routes } from "react-router";
import Home from "./Home";
import Product from "./Product";
import Customer from "./Customer";
import Seller from "./Seller";
import Data from "./Data";
import DataLayout from "./DataLayout";
import About from './About';
import Image from './Image';
import NotFound from './NotFound';
import ProductSearch from './ProductSearch';

createRoot(document.getElementById('root')).render(
    <StrictMode>
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home/>}/>
                <Route path="/about" element={<About />} />
                <Route path="/images/*" element={<Image />} />
                <Route path="/data" element={<DataLayout/>}>
                    <Route index element={<Data/>}/>
                    <Route path="products" element={<Product/>}/>
                    <Route path="products/search" element={<ProductSearch/>}/>
                    <Route path="customers" element={<Customer/>}/>
                    <Route path="sellers" element={<Seller/>}/>
                </Route>
                <Route path='*' element={<NotFound />}/>
            </Routes>
        </BrowserRouter>
    </StrictMode>
)
```

### useLocation

To get the current URL, we could use `window.location.href`, but React Router provides the `useLocation()` hook instead, which returns a `Location` object containing complete information about the current location — path, query, and even the `#fragment`.

> Reference: [react_router.useLocation](https://api.reactrouter.com/v7/functions/react_router.useLocation.html) · [Location](https://api.reactrouter.com/v7/interfaces/react_router.Location.html)

**Task:** display the current location info in the `DataLayout` footer, so it updates on every navigation.

**`src/DataLayout.jsx`**

```jsx
import { NavLink, Outlet, useLocation } from "react-router";
import "./data.css"

export default function DataLayout() {
    const location = useLocation()
    return (
        <>
            <div>
                <h1>This is header</h1>
            </div>
            <div>
                <ul>
                    {/* Reload Page */}
                    {/* <li><a href="/data/products">Products</a></li>
                    <li><a href="/data/customers">Customers</a></li>
                    <li><a href="/data/sellers">Sellers</a></li> */}
                    {/* There is no active link style */}
                    {/* <li><Link to={{
                        pathname: "products",
                        search: "?category=shoes",
                        hash: "#top"
                    }}>Products</Link></li>
                    <li><Link to="customers">Customers</Link></li>
                    <li><Link to="sellers">Sellers</Link></li> */}
                    {/* Add active link style */}
                    <li><NavLink to={{
                        pathname: "products",
                        search: "?category=shoes",
                        hash: "#top"
                    }}>Products</NavLink></li>
                    <li><NavLink to="customers">Customers</NavLink></li>
                    <li><NavLink to="sellers">Sellers</NavLink></li>
                </ul>
            </div>
            <div>
                <Outlet/>
            </div>
            <div>
                <h1>This is Footer</h1>
                <p>Location: {location.pathname}{location.search}{location.hash}</p>
            </div>
        </>
    )
}
```

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **BrowserRouter** | Enables SPA-style routing at the app root | `<BrowserRouter><App /></BrowserRouter>` |
| **Routes / Route** | Maps a URL path to a component | `<Route path="/about" element={<About />} />` |
| **Nested Route** | Shares a path prefix with child routes | `<Route path="/data">...</Route>` |
| **index Route** | Default child for a parent path | `<Route index element={<Data />} />` |
| **Layout Route + Outlet** | Renders a shared layout around child routes | `<Route element={<Layout />}>...</Route>`, `<Outlet />` |
| **Star Segment** | Matches any remaining path, including `/` | `<Route path="/images/*" />`, read via `useParams()['*']` |
| **Not Found Route** | Catch-all for unmatched paths | `<Route path="*" element={<NotFound />} />` (declared last) |
| **Link** | Client-side navigation without a full reload | `<Link to="/about">About</Link>` |
| **NavLink** | Like `Link`, styles itself when active | `<NavLink to="/about">About</NavLink>` |
| **useNavigate** | Navigate programmatically | `const navigate = useNavigate(); navigate('/data')` |
| **useSearchParams** | Read/update query string params | `const [params] = useSearchParams(); params.get('search')` |
| **useLocation** | Read the current path, query, and hash | `const location = useLocation()` |

---

## 💡 Best Practices

**✅ Do This**

- **Wrap the app root in a single `BrowserRouter`** — routing only works for components rendered inside it
- **Use `<Link>` / `<NavLink>` instead of `<a>`** for in-app navigation, so the page never does a full reload
- **Reach for a Layout Route + `<Outlet>`** when several pages share the same header/footer, instead of repeating that markup in every page component
- **Give a parent an `index` route** whenever visiting its base path (e.g. `/data`) should render something by default
- **Place the catch-all `path="*"` route last** in the route list, so it only matches paths nothing else handled
- **Prefer `useNavigate()` for programmatic navigation** (e.g. after a form submits) over manually touching `window.location`

**❌ Avoid This**

- **Using a plain `<a>` tag for internal links** — it triggers a full page reload and defeats the point of an SPA
- **Declaring the `path="*"` Not Found route before more specific routes** — it would swallow those paths since it matches everything
- **Reading the URL with `window.location.href`** inside a component — `useLocation()` re-renders correctly on navigation, a manual read doesn't
- **Forgetting a `key`/param when a star segment or dynamic path should identify different data** — `useParams()` is how that value reaches the component, not a hardcoded prop

> Reference: [reactrouter.com](https://reactrouter.com/) · [api.reactrouter.com/v7](https://api.reactrouter.com/v7/)
