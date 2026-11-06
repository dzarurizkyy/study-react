# ⚛️ React.js Study Case — Contact Management

Study notes for building the **Contact Management** frontend using React, Vite, and React Router — a Single Page Application (SPA) that consumes an already-existing Node.js/Express/Prisma REST API.

Unlike the backend study cases, there is **no database, no ORM, and no global state library** here. Every piece of data lives on the server; the frontend's job is only to (1) render forms, (2) call the right endpoint, (3) carry the session token on every request, and (4) show the result. That narrow scope is deliberate — it makes the actual frontend concerns stand out: routing, controlled forms, effects/lifecycle, and the contract between client and server.

The notes are split into 3 modules — **User Management** → **Contact Management** → **Address Management** — built stage by stage (page by page), plus a long [Bugs Encountered & Fixed](#-bugs-encountered--fixed) section documenting every real integration bug hit along the way, with symptom → root cause → fix → lesson.

---

## 📋 Table of Contents

- [Requirement](#-requirement)
- [Architecture Overview](#-architecture-overview)
- [Backend Setup](#-backend-setup)
- [Frontend Setup](#-frontend-setup)
- [Project Structure](#-project-structure)
- [Application Foundation](#-application-foundation)
  - [index.html — Styling Layer](#indexhtml--styling-layer)
  - [Path Alias `@`](#path-alias-)
  - [Environment Variable](#environment-variable)
  - [The API Layer Convention](#the-api-layer-convention)
  - [Token Management](#token-management)
  - [Alert Helper](#alert-helper)
- [Routing](#-routing)
- [Part 1 — User Management](#-part-1--user-management)
  - [Stage 1: Register User](#stage-1-register-user)
  - [Stage 2: Login User](#stage-2-login-user)
  - [Stage 3: Get Current User](#stage-3-get-current-user)
  - [Stage 4: Update Profile & Password](#stage-4-update-profile--password)
  - [Stage 5: Logout User](#stage-5-logout-user)
- [Part 2 — Contact Management](#-part-2--contact-management)
  - [Stage 1: Create Contact](#stage-1-create-contact)
  - [Stage 2: List & Search Contact](#stage-2-list--search-contact)
  - [Stage 3: Pagination](#stage-3-pagination)
  - [Stage 4: Collapsible Search Form](#stage-4-collapsible-search-form)
  - [Stage 5: Contact Detail](#stage-5-contact-detail)
  - [Stage 6: Update Contact](#stage-6-update-contact)
  - [Stage 7: Delete Contact](#stage-7-delete-contact)
- [Part 3 — Address Management](#-part-3--address-management)
  - [Stage 1: Create Address](#stage-1-create-address)
  - [Stage 2: List Address](#stage-2-list-address)
  - [Stage 3: Update Address](#stage-3-update-address)
  - [Stage 4: Delete Address](#stage-4-delete-address)
- [Running the Project](#-running-the-project)
- [Manual Testing](#-manual-testing)
- [Bugs Encountered & Fixed](#-bugs-encountered--fixed)
- [Known Remaining Issues](#-known-remaining-issues)
- [Quick Reference](#-quick-reference)
- [Key Takeaways](#-key-takeaways)

---

## 🎯 Requirement

UI template used as the starting point: [web-template-contact-management](https://github.com/ProgrammerZamanNow/web-template-contact-management) — a set of static HTML pages that get ported into React components.

The app has 3 features, matching 3 backend modules:

1. **User Management** — register, login, view profile, update profile/password, logout
2. **Contact Management** — contact CRUD + search with pagination
3. **Address Management** — address CRUD, an address always belongs to a contact

**User data**: `username`, `password`, `name`

**Contact data**: `first_name`, `last_name`, `email`, `phone`

**Address data**: `street`, `city`, `province`, `country`, `postal_code`

> **Key Insight:** The ownership chain is `User → Contact → Address`. The frontend never sends a `username` — the backend derives it from the token. Likewise, an address is always addressed through its parent contact's URL (`/api/contacts/:contactId/addresses/...`), never through a standalone `/api/addresses`. That chain shapes the frontend's route structure too: `/dashboard/contacts/:id/addresses/:addressId/edit`.

---

## 🧱 Architecture Overview

```text
┌─────────────────────────────┐         ┌──────────────────────────────┐
│  Frontend (Vite + React)    │         │  Backend (Express + Prisma)  │
│  http://localhost:5173      │         │  http://localhost:3000       │
│                             │         │                              │
│  components/  ← UI + state  │  fetch  │  route → controller →        │
│  lib/api/     ← HTTP calls  │ ──────► │  service → prisma → Postgres │
│  lib/alert.js ← SweetAlert  │ ◄────── │                              │
│  localStorage ← token       │  JSON   │  authMiddleware reads the    │
│                             │         │  Authorization header        │
└─────────────────────────────┘         └──────────────────────────────┘
```

Two things flow across that boundary, and getting either wrong is where most of the bugs came from:

| Direction | Contract |
| --- | --- |
| Frontend → Backend | `Authorization: <token>` header (a raw UUID string, **no** `Bearer` prefix) |
| Backend → Frontend | Success: `{ "data": ... }` · Failure: `{ "errors": "..." }` · Search also adds `paging` |

Because they run on different ports, the browser treats every call as cross-origin — hence the CORS setup below.

---

## 🏗️ Backend Setup

This class does not build a backend. It reuses the Node.js one so both ends are the same language: [belajar-nodejs-restful-api](https://github.com/ProgrammerZamanNow/belajar-nodejs-restful-api).

### Clone & Install

```bash
git clone git@github.com:ProgrammerZamanNow/belajar-nodejs-restful-api.git
cd belajar-nodejs-restful-api
npm install
npx prisma generate
```

### Setup Database

```sql
create database belajar_reactjs_contact_management;
```

### Database Migration

Set the connection string in `.env`, then create all tables:

```bash
npx prisma migrate dev
```

### Setup CORS

The frontend (Vite, port `5173`) and backend (Express, port `3000`) are different origins, so the browser blocks the requests by default with a CORS (Cross Origin Resource Sharing) error. The backend has to opt in:

```bash
npm install cors
```

**`src/application/web.js`**

```javascript
import express from "express";
import { publicRouter } from "../route/public-api.js";
import { errorMiddleware } from "../middleware/error-middleware.js";
import { userRouter } from "../route/api.js";
import cors from "cors";

export const web = express();
web.use(cors());          // ← must come before the routers
web.use(express.json());

web.use(publicRouter);    // no token required
web.use(userRouter);      // token required (authMiddleware)

web.use(errorMiddleware); // last — turns thrown errors into JSON
```

> ⚠️ **Note:** `web.use(cors())` allows *all* origins. That's fine for local study, but in production it should be restricted to the actual frontend domain.

### Understanding the Backend Response Shape

Reading these three backend files first saves a lot of frontend guesswork — they define exactly what the React code will receive.

**`src/controller/user-controller.js`** — every success is wrapped in `data`:

```javascript
const register = async (req, res, next) => {
    try {
        const result = await userService.register(req.body);
        res.status(200).json({
            data: result          // ← always an envelope, never the bare object
        });
    } catch (e) {
        next(e);
    }
}
```

**`src/middleware/error-middleware.js`** — every failure is wrapped in `errors`:

```javascript
const errorMiddleware = async (err, req, res, next) => {
    if (!err) { next(); return; }

    if (err instanceof ResponseError) {
        res.status(err.status).json({ errors: err.message }).end();
    } else {
        res.status(500).json({ errors: err.message }).end();
    }
}
```

**`src/middleware/auth-middleware.js`** — how the token is validated:

```javascript
export const authMiddleware = async (req, res, next) => {
    const token = req.get('Authorization');   // ← the raw header value, as-is
    if (!token) {
        res.status(401).json({ errors: "Unauthorized" }).end();
    } else {
        const user = await prismaClient.user.findFirst({
            where: { token: token }           // ← compared directly to the DB column
        });
        if (!user) {
            res.status(401).json({ errors: "Unauthorized" }).end();
        } else {
            req.user = user;
            next();
        }
    }
}
```

> **Key Insight:** `authMiddleware` compares the header value **directly** against the `token` column, which holds a bare `uuid` generated at login. There is no `Bearer` scheme parsing, no `split(' ')[1]`. This single detail is the source of [Bug #3](#3-authorization-bearer-token-vs-a-raw-token-backend) — the frontend must send the token *unprefixed*.

### Running the Backend

```bash
node src/main.js
# App start → listening on http://localhost:3000
```

A quick health check endpoint exists on the public router: `GET /ping`.

---

## ⚛️ Frontend Setup

### Creating the Project

```bash
npm create vite@latest study-reatjs-contact-management -- --template react
cd study-reatjs-contact-management
npm install
```

### Adding React Router

Routing for the SPA — this project uses the modern `react-router` package (v7+/v8), where `BrowserRouter`, `Routes`, `Route`, `Link`, `Outlet`, `useNavigate`, and `useParams` are all imported from `react-router` directly (not `react-router-dom`).

```bash
npm install react-router
```

### Adding React Use

A popular hooks library. This project uses two of its hooks heavily:

- `useLocalStorage(key, initial)` — a `useState`-like API that persists to `localStorage`
- `useEffectOnce(fn)` — `useEffect(fn, [])`, but with the intent spelled out in the name

```bash
npm install react-use
```

> Reference: [github.com/streamich/react-use](https://github.com/streamich/react-use)

### Adding SweetAlert2

Nicer alert/confirm dialogs than the browser's built-in `alert()` / `confirm()` — and, importantly, promise-based, so `await`ing a confirm dialog reads naturally.

```bash
npm install sweetalert2
```

> Reference: [sweetalert2.github.io](https://sweetalert2.github.io/)

### Final Dependency List

**`package.json`**

```json
{
  "dependencies": {
    "react": "^19.2.8",
    "react-dom": "^19.2.8",
    "react-router": "^8.3.0",
    "react-use": "^17.6.1",
    "sweetalert2": "^11.26.25"
  }
}
```

Note what is **not** there: no Redux/Zustand/Jotai, no axios (native `fetch` is enough), no Tailwind package (it's loaded via CDN — see below), no form library.

---

## 🗂️ Project Structure

```text
study-reatjs-contact-management/
├── index.html                     # Tailwind CDN, fonts, custom CSS, #root
├── vite.config.js                 # react plugin + "@" path alias
├── jsconfig.json                  # editor: resolve "@", enable JS checking
├── .env                           # VITE_API_PATH
└── src/
    ├── main.jsx                   # createRoot + all route definitions
    ├── components/
    │   ├── Layout.jsx             # public shell (centered card) → <Outlet />
    │   ├── DashboardLayout.jsx    # authenticated shell (navbar) → <Outlet />
    │   ├── User/
    │   │   ├── UserRegister.jsx
    │   │   ├── UserLogin.jsx
    │   │   ├── UserProfile.jsx    # two forms: name, password
    │   │   └── UserLogout.jsx     # headless — no UI, just an effect
    │   ├── Contact/
    │   │   ├── ContactList.jsx    # search + pagination + delete
    │   │   ├── ContactCreate.jsx
    │   │   ├── ContactDetail.jsx  # contact info + its address list
    │   │   └── ContactEdit.jsx
    │   └── Address/
    │       ├── AddressCreate.jsx
    │       └── AdressEdit.jsx     # (filename typo kept as-is in the project)
    └── lib/
        ├── alert.js               # SweetAlert2 wrappers
        └── api/
            ├── UserApi.js
            ├── ContactApi.js
            └── AddressApi.js
```

> **Key Insight:** There is no `pages/` vs `components/` split and no `store/` folder. Routing is flat enough that React Router alone handles navigation, and state is either local to a component (`useState`) or the single session token (`localStorage`). The only real "layer" here is `lib/api/` — isolating every `fetch` call keeps components free of URL strings and header details.

---

## ⚙️ Application Foundation

Before any feature, these pieces get set up once and are used by every component.

### index.html — Styling Layer

The UI template is Tailwind-based, but Tailwind is pulled from a CDN rather than installed as a build dependency — so there's no PostCSS config, no `tailwind.config.js` file, and no build step for CSS. Configuration is inlined in a `<script>` tag instead.

**`index.html`**

```html
<head>
  <meta charset="UTF-8">
  <title>Contact Management</title>

  <!-- Tailwind CSS via CDN -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- Google Fonts: Poppins -->
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">

  <!-- Font Awesome — every <i className="fas fa-..."> icon comes from here -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: { sans: ['Poppins', 'sans-serif'] },
          boxShadow: {
            'custom': '0 10px 25px -5px rgba(0,0,0,.1), 0 10px 10px -5px rgba(0,0,0,.04)',
          },
        }
      }
    }
  </script>

  <style>
    .bg-gradient   { background: linear-gradient(135deg, #1e3a8a 0%, #1e40af 100%); }
    .animate-fade-in { animation: fadeIn .5s ease-in-out; }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(-10px); }
      to   { opacity: 1; transform: translateY(0); }
    }
    .card-hover        { transition: all .3s ease; }
    .card-hover:hover  { transform: translateY(-5px); box-shadow: 0 15px 30px -5px rgba(0,0,0,.3); }
  </style>
</head>

<body>
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
```

This explains three class names that appear all over the components but are **not** Tailwind utilities: `bg-gradient`, `animate-fade-in`, `card-hover`, plus the custom `shadow-custom` shadow registered through `tailwind.config`.

### Path Alias `@`

Two files are needed — one for the bundler, one for the editor.

**`vite.config.js`** — so Vite can actually resolve `@/lib/alert` at build time:

```javascript
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
})
```

**`jsconfig.json`** — so the editor's language server understands the same alias (autocomplete, go-to-definition, auto-import):

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "allowJs": true,
    "checkJs": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

> **Key Insight:** `allowJs` and `checkJs` do different jobs. `allowJs: true` is enough for **autocomplete and auto-import while typing**. But the **Quick Fix lightbulb (`Cmd+.`) only appears where there is a reported diagnostic** — and in a plain `.js`/`.jsx` file, TypeScript only reports diagnostics when `checkJs: true`. With `checkJs: false`, a missing import produces no red squiggle, therefore no Quick Fix, even though typing the same symbol *does* offer an auto-import suggestion. Two different editor features, two different requirements.

### Environment Variable

**`.env`**

```bash
VITE_API_PATH=http://localhost:3000
```

Only variables prefixed with `VITE_` are exposed to client code by Vite. Every API function reads it:

```javascript
fetch(`${import.meta.env.VITE_API_PATH}/api/users`, { ... })
```

> ⚠️ **Note:** anything in a `VITE_`-prefixed variable ends up in the browser bundle in plain text. Base URLs are fine; secrets never belong here.

### The API Layer Convention

Every function in `src/lib/api/*.js` follows the same three rules:

1. Build the URL from `import.meta.env.VITE_API_PATH`
2. Attach `Authorization: token` when the endpoint requires auth
3. **Return the raw `Response` object** — never call `.json()` inside the API layer

**`src/lib/api/UserApi.js`** (complete)

```javascript
export const userRegister = async ({ username, password, name }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password, name }),
  });
};

export const userLogin = async ({ username, password }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password }),
  });
};

export const userDetail = async (token) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      Authorization: token,
    },
  });
};

export const userUpdateProfile = async (token, { name }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      Authorization: token,
    },
    body: JSON.stringify({ name }),
  });
};

export const userUpdatePassword = async (token, { password }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      Authorization: token,
    },
    body: JSON.stringify({ password }),
  });
};

export const userLogout = async (token) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/logout`, {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json',
      Authorization: token,
    },
  });
};
```

Which gives every calling component the exact same three-line shape:

```javascript
const response = await someApiCall(...);      // Response
const responseBody = await response.json();   // { data } or { errors }

if (response.status === 200) { /* success */ } else { /* alertError */ }
```

> **Key Insight:** Returning the raw `Response` (instead of parsed JSON) is what makes `response.status` available to the caller. The backend signals success/failure through the **HTTP status**, and the body carries either `data` or `errors` — so the component needs both halves. Break this convention in even one function and the caller crashes: see [Bug #6](#6-one-api-function-silently-parsing-the-body-itself).

Note also that `userUpdateProfile` and `userUpdatePassword` hit the *same* endpoint (`PATCH /api/users/current`) — the backend's `updateUserValidation` marks both `name` and `password` optional, so each function sends only its own field. Splitting them into two functions keeps the two independent forms on the profile page independent.

### Token Management

The backend represents a session as a plain `token` column on the `users` row — a `uuid` regenerated on every login. The frontend stores it in the browser's `localStorage` so it survives refreshes and can be attached to later requests.

```javascript
// write it (login)
const [, setToken] = useLocalStorage('token', '');
setToken(responseBody.data.token);

// read it (every authenticated page)
const [token] = useLocalStorage('token', '');

// clear it (logout)
const [token, setToken] = useLocalStorage('token', '');
setToken('');
```

`useLocalStorage` from `react-use` returns a `[value, setValue, remove]` tuple that behaves like `useState`, but reads the initial value from `localStorage` and writes back on every update.

> Reference: [developer.mozilla.org — Window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)

### Alert Helper

**`src/lib/alert.js`** (complete)

```javascript
import Swal from 'sweetalert2';

export const alertSuccess = async (message) => {
  return Swal.fire({ icon: 'success', title: 'Success', text: message });
};

export const alertError = async (message) => {
  return Swal.fire({ icon: 'error', title: 'Error', text: message });
};

export const alertConfirm = async (message) => {
  const result = await Swal.fire({
    icon: 'question',
    title: 'Are you sure?',
    text: message,
    showCancelButton: true,
    confirmButtonText: 'Yes',
    confirmButtonColor: '#3085d6',
    cancelButtonColor: '#d33',
  });

  return result.isConfirmed;   // ← unwrap to a plain boolean
};
```

> **Key Insight:** `alertConfirm` returns `result.isConfirmed` rather than SweetAlert's full result object. That small mapping is what lets every delete handler read like plain language: `if (await alertConfirm('Are you sure?')) { ... }`. Wrapping a library at its boundary — instead of calling `Swal.fire` in twelve components — also means the dialog styling can change in exactly one place.

---

## 🧭 Routing

**`src/main.jsx`** (complete)

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { BrowserRouter, Route, Routes } from 'react-router';
import Layout from './components/Layout';
import UserRegister from './components/User/UserRegister';
import UserLogin from './components/User/UserLogin';
import DashboardLayout from './components/DashboardLayout';
import UserProfile from './components/User/UserProfile';
import UserLogout from './components/User/UserLogout';
import ContactCreate from './components/Contact/ContactCreate';
import ContactList from './components/Contact/ContactList';
import ContactEdit from './components/Contact/ContactEdit';
import ContactDetail from './components/Contact/ContactDetail';
import AddressCreate from './components/Address/AddressCreate';
import AddressEdit from './components/Address/AdressEdit';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        <Route>
          <Route element={<Layout />}>
            <Route path={'register'} element={<UserRegister />}></Route>
            <Route path={'login'} element={<UserLogin />}></Route>
          </Route>
        </Route>

        <Route path={'dashboard'} element={<DashboardLayout />}>
          <Route path={'users'}>
            <Route path="profile" element={<UserProfile />}></Route>
            <Route path="logout" element={<UserLogout />}></Route>
          </Route>

          <Route path={'contacts'}>
            <Route index element={<ContactList />} />
            <Route path={'create'} element={<ContactCreate />} />
            <Route path={':id'}>
              <Route index element={<ContactDetail />}></Route>
              <Route path={'edit'} element={<ContactEdit />}></Route>
              <Route path={'addresses'}>
                <Route path={'create'} element={<AddressCreate />}></Route>
                <Route path={':addressId/edit'} element={<AddressEdit />}></Route>
              </Route>
            </Route>
          </Route>
        </Route>
      </Routes>
    </BrowserRouter>
  </StrictMode>,
);
```

### URL → Component Map

| URL | Layout | Component |
| --- | --- | --- |
| `/register` | `Layout` | `UserRegister` |
| `/login` | `Layout` | `UserLogin` |
| `/dashboard/users/profile` | `DashboardLayout` | `UserProfile` |
| `/dashboard/users/logout` | `DashboardLayout` | `UserLogout` |
| `/dashboard/contacts` | `DashboardLayout` | `ContactList` (index route) |
| `/dashboard/contacts/create` | `DashboardLayout` | `ContactCreate` |
| `/dashboard/contacts/:id` | `DashboardLayout` | `ContactDetail` (index route) |
| `/dashboard/contacts/:id/edit` | `DashboardLayout` | `ContactEdit` |
| `/dashboard/contacts/:id/addresses/create` | `DashboardLayout` | `AddressCreate` |
| `/dashboard/contacts/:id/addresses/:addressId/edit` | `DashboardLayout` | `AddressEdit` |

Three routing concepts do all the work here:

**1. Nested paths compose.** A child `<Route path="edit">` inside `<Route path=":id">` inside `<Route path="contacts">` inside `<Route path="dashboard">` produces `/dashboard/contacts/:id/edit`. Only the leaf declares an `element`; the middle `<Route path={'contacts'}>` and `<Route path={':id'}>` are pure grouping nodes.

**2. `index` routes** render at the parent's exact path — `<Route index element={<ContactList />} />` is what makes `/dashboard/contacts` (with nothing after it) show the list.

**3. Layout routes need `<Outlet />`.** A route with an `element` and children renders its element as a *shell*, and React Router injects the matched child wherever `<Outlet />` appears inside it.

**`src/components/Layout.jsx`** — the public shell, a centered card on a gradient background:

```jsx
import { Outlet } from "react-router";

export default function Layout() {
  return (
    <div className="bg-gradient-to-br from-gray-900 to-gray-800 min-h-screen flex items-center justify-center p-4">
      <Outlet />
    </div>
  );
}
```

**`src/components/DashboardLayout.jsx`** — the authenticated shell, with a navbar and footer around the content:

```jsx
import { Link, Outlet } from "react-router";

export default function DashboardLayout() {
  return (
    <div className="bg-gradient-to-br from-gray-900 to-gray-800 min-h-screen flex flex-col">
      {/* Header */}
      <header className="bg-gradient shadow-lg">
        <div className="container mx-auto px-4 py-4 flex justify-between items-center">
          <Link to="/dashboard/contacts" className="flex items-center ...">
            <i className="fas fa-address-book text-white text-2xl mr-3"></i>
            <div className="text-white font-bold text-xl">Contact Management</div>
          </Link>

          <nav>
            <ul className="flex space-x-6">
              <li><Link to="/dashboard/users/profile">Profile</Link></li>
              <li><Link to="/dashboard/users/logout">Logout</Link></li>
            </ul>
          </nav>
        </div>
      </header>

      {/* Main */}
      <main className="container mx-auto px-4 py-8 flex-grow">
        <Outlet />          {/* ← child route renders HERE */}

        <div className="mt-10 mb-6 text-center text-gray-400 text-sm animate-fade-in">
          <p>© 2025 Contact Management. All rights reserved.</p>
        </div>
      </main>
    </div>
  );
}
```

> **Key Insight:** Navigation always uses `<Link to="...">`, never `<a href="...">`. An anchor tag triggers a **full page reload** — the whole React app is torn down and rebooted, losing all in-memory state and defeating the point of an SPA. `<Link>` swaps the rendered route client-side and only updates the URL. (The original HTML template used `<a href="detail_contact.html">`, which all had to be converted during the port.)
>
> ⚠️ **Note:** These routes have **no auth guard**. `/dashboard/*` renders for anyone; an unauthenticated visitor simply gets a `401` from the first API call and an error alert. A production app would wrap the dashboard branch in a component that redirects to `/login` when `localStorage` has no token — see [Known Remaining Issues](#-known-remaining-issues).

---

## 👤 Part 1 — User Management

### User API Spec

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| POST | `/api/users` | - | Register a new user |
| POST | `/api/users/login` | - | Login, receive a token |
| GET | `/api/users/current` | ✅ | Get the logged-in user's data |
| PATCH | `/api/users/current` | ✅ | Update name and/or password |
| DELETE | `/api/users/logout` | ✅ | Clear the token (logout) |

**Register** — `POST /api/users`

```json
// Request
{ "username": "dzarurizkyy", "password": "dzaru2024", "name": "Dzaru Rizky" }
```

```json
// Response 200
{ "data": { "username": "dzarurizkyy", "name": "Dzaru Rizky" } }
```

```json
// Response 400
{ "errors": "Username already exists" }
```

**Login** — `POST /api/users/login`

```json
// Request
{ "username": "dzarurizkyy", "password": "dzaru2024" }
```

```json
// Response 200 — the token the frontend must store
{ "data": { "token": "3fa85f64-5717-4562-b3fc-2c963f66afa6" } }
```

```json
// Response 401
{ "errors": "Username or password wrong" }
```

**Get Current** — `GET /api/users/current` — Header: `Authorization: <token>`

```json
{ "data": { "username": "dzarurizkyy", "name": "Dzaru Rizky" } }
```

**Update** — `PATCH /api/users/current` — both fields optional

```json
// Request
{ "name": "New Name" }
```

**Logout** — `DELETE /api/users/logout`

```json
{ "data": "OK" }
```

---

### Stage 1: Register User

The first page, and the first taste of the pattern every later form follows: controlled inputs + an async submit handler.

**`src/lib/api/UserApi.js`**

```javascript
export const userRegister = async ({ username, password, name }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password, name }),
  });
};
```

**`src/components/User/UserRegister.jsx`**

```jsx
import { alertError, alertSuccess } from '@/lib/alert';
import { userRegister } from '@/lib/api/UserApi';
import { useState } from 'react';
import { Link, useNavigate } from 'react-router';

export default function UserRegister() {
  const [username, setUsername] = useState('');
  const [name, setName] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const navigate = useNavigate();

  async function handleSubmit(e) {
    e.preventDefault();                       // stop the browser's native form POST

    if (password !== confirmPassword) {       // client-side check the API doesn't do
      await alertError('Password do not match');
      return;
    }

    const response = await userRegister({ username, password, name });
    const responseBody = await response.json();

    if (response.status === 200) {
      await alertSuccess('User registered successfully');
      navigate('/login');                     // client-side redirect, no page reload
    } else {
      await alertError(responseBody.errors);
      return;
    }
  }

  return (
    <div className="animate-fade-in bg-gray-800 ... w-full max-w-md">
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          id="username"
          value={username}                              /* state → input */
          onChange={(e) => setUsername(e.target.value)} /* input → state */
          required
        />
        {/* ...name, password, confirm_password inputs, same pattern... */}

        <button type="submit">Register</button>
      </form>

      <div className="text-center text-sm text-gray-400">
        Already have an account? <Link to="/login">Sign in</Link>
      </div>
    </div>
  );
}
```

Four things are worth naming explicitly, because every other form in the project repeats them:

- **`e.preventDefault()`** — without it the browser performs its own form submission and reloads the page, killing the React app mid-request.
- **Controlled inputs** — `value={username}` + `onChange={...setUsername(e.target.value)}` makes React state the single source of truth. Forget the `onChange` and the field becomes read-only; forget `value` and React can't read what the user typed.
- **`confirmPassword` is frontend-only** — it is never sent to the API. The backend has no concept of a confirmation field, so this validation has to live here.
- **`useNavigate()`** — the programmatic equivalent of `<Link>`, for redirecting *after* an async result rather than on a click.

> **Key Insight:** `await alertSuccess(...)` before `navigate(...)` is intentional. SweetAlert's `Swal.fire()` returns a promise that resolves when the dialog is **dismissed**, so awaiting it means "show the message, wait for the user to acknowledge, *then* redirect." Dropping the `await` would navigate away while the dialog is still opening.

---

### Stage 2: Login User

Login is where the session begins: the response carries a token, and everything after this point depends on it being stored correctly.

**`src/lib/api/UserApi.js`**

```javascript
export const userLogin = async ({ username, password }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password }),
  });
};
```

**`src/components/User/UserLogin.jsx`**

```jsx
import { alertError, alertSuccess } from '@/lib/alert';
import { userLogin } from '@/lib/api/UserApi';
import { useState } from 'react';
import { Link, useNavigate } from 'react-router';
import { useLocalStorage } from 'react-use';

export default function UserLogin() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const navigate = useNavigate();
  const [, setToken] = useLocalStorage('token', '');   // only the setter is needed here

  async function handleSubmit(e) {
    e.preventDefault();

    const response = await userLogin({ username, password });
    const responseBody = await response.json();

    if (response.status === 200) {
      const token = responseBody.data.token;   // ← note the `data` envelope
      setToken(token);                         // ← persisted to localStorage
      await alertSuccess('Login success');
      navigate('/dashboard/contacts');
    } else {
      await alertError(responseBody.errors);
    }
  }

  return (/* ...username + password form, same controlled-input pattern... */);
}
```

`const [, setToken] = useLocalStorage(...)` — the empty first slot is array-destructuring skipping the current value, because this component only writes the token, never reads it.

> **Key Insight:** Because the token lives in `localStorage` and not in React state, it survives a page refresh **and** is readable from any component without prop-drilling or a context provider. That's the whole reason this app needs no global state library: the only genuinely global value is one string, and the browser already provides a global place to keep it.

---

### Stage 3: Get Current User

The first page that *reads* data on mount instead of reacting to a click — which introduces the effect pattern used by every detail/edit page after it.

**`src/lib/api/UserApi.js`**

```javascript
export const userDetail = async (token) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      Authorization: token,          // ← raw token, no "Bearer " prefix
    },
  });
};
```

**`src/components/User/UserProfile.jsx`** (fetch part)

```jsx
import { useEffectOnce, useLocalStorage } from 'react-use';

export default function UserProfile() {
  const [name, setName] = useState('');
  const [token] = useLocalStorage('token', '');

  async function fetchUserDetail() {
    const response = await userDetail(token);
    const responseBody = await response.json();

    if (response.status === 200) {
      setName(responseBody.data.name);   // ← prefills the "Full Name" input
    } else {
      alertError(responseBody.errors);
    }
  }

  useEffectOnce(() => {
    fetchUserDetail();
  });

  // ...forms below...
}
```

Two details that recur everywhere:

- **`useEffectOnce` instead of `useEffect(fn, [])`** — functionally identical, but the name states the intent, and (as it happens) it also sidesteps a lint rule described in [Bug #8](#8-a-lint-rule-that-only-recognizes-reacts-own-effect-hooks).
- **The effect callback is not `async`.** `fetchUserDetail` is declared separately and *called* from the effect. An effect's return value is reserved for its cleanup function, and an `async` callback returns a promise — React would misinterpret it.

> **Key Insight:** Fetching into the same `name` state that the edit form is bound to is what makes the profile form "prefilled". There is no separate "loaded data" vs "form data" — the fetch simply seeds the controlled input's state, and the user's typing takes it from there.

---

### Stage 4: Update Profile & Password

One page, two independent forms, two separate submit handlers — both hitting the same `PATCH /api/users/current` endpoint with a different field.

**`src/lib/api/UserApi.js`**

```javascript
export const userUpdateProfile = async (token, { name }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json', Authorization: token },
    body: JSON.stringify({ name }),
  });
};

export const userUpdatePassword = async (token, { password }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/current`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json', Authorization: token },
    body: JSON.stringify({ password }),
  });
};
```

**`src/components/User/UserProfile.jsx`** (submit handlers)

```jsx
async function handleSubmitName(e) {
  e.preventDefault();

  const response = await userUpdateProfile(token, { name });
  const responseBody = await response.json();

  if (response.status === 200) {
    alertSuccess(responseBody.message);
  } else {
    alertError(responseBody.errors);
  }
}

async function handleSubmitPassword(e) {
  e.preventDefault();

  if (newPassword !== confirmPassword) {
    alertError('Password and confirmation do not match');
    return;
  }

  const response = await userUpdatePassword(token, { password: newPassword });
  const responseBody = await response.json();

  if (response.status === 200) {
    alertSuccess(responseBody.message);
    setNewPassword('');        // clear the fields — never leave a password on screen
    setConfirmPassword('');
  } else {
    alertError(responseBody.errors);
  }
}
```

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 gap-8">
  <form onSubmit={handleSubmitName}>     {/* Form 1: Edit Profile  */}</form>
  <form onSubmit={handleSubmitPassword}> {/* Form 2: Change Password */}</form>
</div>
```

Because `PATCH` treats both fields as optional, sending only `{ name }` leaves the password untouched and vice-versa — which is exactly what allows two separate forms to share one endpoint without stepping on each other.

> ⚠️ **Note:** `alertSuccess(responseBody.message)` reads a field the backend never sends (its update controller returns `{ data: {...} }`). The dialog therefore shows an empty body. See [Known Remaining Issues](#-known-remaining-issues).

---

### Stage 5: Logout User

The one component with **no UI at all** — it exists purely to run a side effect and redirect.

**`src/lib/api/UserApi.js`**

```javascript
export const userLogout = async (token) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/users/logout`, {
    method: 'DELETE',
    headers: { 'Content-Type': 'application/json', Authorization: token },
  });
};
```

**`src/components/User/UserLogout.jsx`** (complete)

```jsx
import { useEffectOnce, useLocalStorage } from 'react-use';
import { userLogout } from '../../lib/api/UserApi';
import { useNavigate } from 'react-router';
import { alertError } from '@/lib/alert';

export default function UserLogout() {
  const [token, setToken] = useLocalStorage('token', '');
  const navigate = useNavigate();

  async function handleLogout() {
    const response = await userLogout(token);
    const responseBody = await response.json();

    if (response.status === 200) {
      setToken('');            // clear the browser-side session
      await navigate('/login');
    } else {
      await alertError(responseBody.errors);
    }
  }

  useEffectOnce(() => {
    handleLogout();
  });

  return <></>;                // renders nothing — it's a route-as-action
}
```

> **Key Insight:** Making logout a *route* (`/dashboard/users/logout`) rather than an `onClick` handler means the navbar's Logout entry can be a plain `<Link>`, identical in markup to Profile. The component returns an empty fragment because its entire purpose is the effect: invalidate the token server-side, clear it client-side, redirect. Both halves matter — clearing only `localStorage` would leave a still-valid token in the database.

With that, User Management is complete: **Register → Login → Get Current → Update → Logout**.

---

## 📇 Part 2 — Contact Management

### Contact API Spec

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| POST | `/api/contacts` | ✅ | Create a new contact |
| GET | `/api/contacts/:contactId` | ✅ | Get one contact's detail |
| PUT | `/api/contacts/:contactId` | ✅ | Update a contact |
| DELETE | `/api/contacts/:contactId` | ✅ | Delete a contact |
| GET | `/api/contacts?name=&email=&phone=&page=&size=` | ✅ | Search + pagination |

The search endpoint is the only one that returns a second top-level key besides `data`:

```json
{
  "data": [ { "id": 1, "first_name": "...", "last_name": "...", "email": "...", "phone": "..." } ],
  "paging": { "page": 1, "total_item": 7, "total_page": 4 }
}
```

> ⚠️ **Note:** the backend's `searchContactValidation` defaults `size` to **2**, not 10. With only a handful of contacts the list already spans several pages — convenient for testing pagination, surprising if unexpected.

---

### Stage 1: Create Contact

**`src/lib/api/ContactApi.js`**

```javascript
export const contactCreate = async (token, { first_name, last_name, email, phone }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/contacts`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', Authorization: token },
    body: JSON.stringify({ first_name, last_name, email, phone }),
  });
};
```

**`src/components/Contact/ContactCreate.jsx`**

```jsx
export default function ContactCreate() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');
  const [phone, setPhone] = useState('');
  const navigate = useNavigate();
  const [token] = useLocalStorage('token', '');

  async function handleSubmit(e) {
    e.preventDefault();

    const response = await contactCreate(token, {
      first_name: firstName,     // camelCase state → snake_case API field
      last_name: lastName,
      email: email,
      phone: phone,
    });

    const responseBody = await response.json();

    if (response.status === 200) {
      navigate('/dashboard/contacts');
      await alertSuccess('Contact created');
    } else {
      alertError(responseBody.errors);
    }
  }

  return (/* ...4 controlled inputs + Cancel <Link> + submit button... */);
}
```

> **Key Insight:** The frontend uses `camelCase` state names (`firstName`) while the API speaks `snake_case` (`first_name`). The mapping happens in exactly one place — the object literal passed to the API function — so the JSX stays idiomatic React and the wire format stays whatever the backend expects. Doing this conversion inside `lib/api/` instead would work too; the important part is that it happens at one boundary, not scattered.

Notice the ordering here is `navigate(...)` **then** `await alertSuccess(...)` — the reverse of `UserRegister`. Both work; this variant redirects immediately and shows the confirmation on the destination page.

---

### Stage 2: List & Search Contact

The most involved page in the project. Search filters, pagination, and delete all live in `ContactList.jsx`.

**`src/lib/api/ContactApi.js`**

```javascript
export const contactList = async (token, { name, phone, email, page }) => {
  const url = new URL(`${import.meta.env.VITE_API_PATH}/api/contacts`);

  if (name)  url.searchParams.set('name', name);
  if (email) url.searchParams.set('email', email);
  if (phone) url.searchParams.set('phone', phone);
  if (page)  url.searchParams.set('page', page);

  return await fetch(url, {
    method: 'GET',
    headers: { 'Content-Type': 'application/json', Authorization: token },
  });
};
```

Using the `URL` + `searchParams` API instead of manual string concatenation means: no `?` vs `&` bookkeeping, automatic percent-encoding of values (a search for `a&b` won't break the query string), and empty filters are simply never added — so a blank search sends `/api/contacts` with no params at all rather than `?name=&email=&phone=`.

**`src/components/Contact/ContactList.jsx`** (state + fetch)

```jsx
export default function ContactList() {
  const [token] = useLocalStorage('token', '');

  // search filters
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [phone, setPhone] = useState('');

  // pagination
  const [page, setPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);

  // data + refetch trigger
  const [contacts, setContacts] = useState([]);
  const [reload, setReload] = useState(false);

  async function fetchContacts() {
    const response = await contactList(token, { name, phone, email, page });
    const responseBody = await response.json();

    if (response.status === 200) {
      setContacts(responseBody.data);                    // the array of contacts
      setTotalPages(responseBody.paging.total_page);     // for the page buttons
    } else {
      await alertError(responseBody.errors);
    }
  }

  async function handleSearchContacts(e) {
    e.preventDefault();
    setReload(true);         // don't fetch here — just ask for a refetch
  }

  useEffect(() => {
    if (reload) {
      fetchContacts().then(() => console.log('Contacts fetched'));
      setReload(false);      // consume the flag so it fires exactly once
    }
  }, [reload]);
```

**Rendering the list:**

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {/* A permanent "Create New Contact" card, always first in the grid */}
  <div className="... border-2 border-dashed ...">
    <Link to="/dashboard/contacts/create" className="block p-6 h-full">
      <h2>Create New Contact</h2>
    </Link>
  </div>

  {contacts.map((contact) => (
    <div key={`contact-${contact.id}`} className="...">
      <Link to={`/dashboard/contacts/${contact.id}`}>
        <h2>{contact.first_name} {contact.last_name}</h2>
        <span>{contact.email}</span>
        <span>{contact.phone}</span>
      </Link>

      <Link to={`/dashboard/contacts/${contact.id}/edit`}>Edit</Link>
      <button onClick={() => handleContactDelete(contact.id)}>Delete</button>
    </div>
  ))}
</div>
```

- **`key={`contact-${contact.id}`}`** — a stable, unique key per row. Using the array index instead would make React reuse the wrong DOM nodes when the list is filtered or a row is deleted.
- **`onClick={() => handleContactDelete(contact.id)}`** — an arrow wrapper, because `onClick={handleContactDelete(contact.id)}` would *call* the function during render instead of on click.

> **Key Insight:** This page never calls `fetchContacts()` directly from an event handler. Search, page change, and delete all just set `reload = true`, and the single `useEffect` performs the fetch. That indirection is what guarantees the request always sees the **latest** `page` / `name` / `email` / `phone` values — the fix for [Bug #7](#7-stale-state-read-right-after-setstate).

---

### Stage 3: Pagination

**`ContactList.jsx`** (pagination logic)

```jsx
function getPages() {
  const pages = [];
  for (let i = 1; i <= totalPages; i++) {
    pages.push(i);
  }
  return pages;                 // [1, 2, 3, ...] derived from the API's total_page
}

async function handlePageChange(pageValue) {
  setPage(pageValue);           // schedule the state update...
  setReload(true);              // ...and let the effect fetch once it's applied
}
```

```jsx
<nav className="flex items-center space-x-3 ...">
  {page > 1 && (
    <button onClick={() => handlePageChange(page - 1)}>Previous</button>
  )}

  {getPages().map((value) => {
    if (value === page) {
      return (                                  /* active page — gradient style */
        <button key={value} onClick={() => handlePageChange(value)}
                className="bg-gradient text-white ...">{value}</button>
      );
    }
    return (                                    /* inactive page — grey style */
      <button key={value} onClick={() => handlePageChange(value)}
              className="bg-gray-700 text-gray-300 ...">{value}</button>
    );
  })}

  {page < totalPages && (
    <button onClick={() => handlePageChange(page + 1)}>Next</button>
  )}
</nav>
```

`totalPages` is never computed on the frontend — it comes straight from the backend's `paging.total_page`, which already accounts for the filters that were applied. `getPages()` just turns that count into a `[1..n]` array to map over, and `{page > 1 && ...}` / `{page < totalPages && ...}` hide Previous/Next at the boundaries.

---

### Stage 4: Collapsible Search Form

The search panel collapses and expands via a chevron button. This is the one place in the project doing **direct DOM manipulation** — inherited from the original static template, and kept as an exercise in wiring imperative DOM code into a React lifecycle.

```jsx
useEffectOnce(() => {
  const toggleButton = document.getElementById('toggleSearchForm');
  const searchFormContent = document.getElementById('searchFormContent');
  const toggleIcon = document.getElementById('toggleSearchIcon');

  // start collapsed, with a transition for the open/close animation
  searchFormContent.style.transition =
    'max-height 0.3s ease-in-out, opacity 0.3s ease-in-out, margin 0.3s ease-in-out';
  searchFormContent.style.overflow = 'hidden';
  searchFormContent.style.maxHeight = '0px';
  searchFormContent.style.opacity = '0';
  searchFormContent.style.marginTop = '0';

  function toggleSearchForm() {
    if (searchFormContent.style.maxHeight !== '0px') {
      searchFormContent.style.maxHeight = '0px';          // hide
      searchFormContent.style.opacity = '0';
      searchFormContent.style.marginTop = '0';
      toggleIcon.classList.remove('fa-chevron-up');
      toggleIcon.classList.add('fa-chevron-down');
    } else {
      searchFormContent.style.maxHeight = searchFormContent.scrollHeight + 'px';
      searchFormContent.style.opacity = '1';              // show
      searchFormContent.style.marginTop = '1rem';
      toggleIcon.classList.remove('fa-chevron-down');
      toggleIcon.classList.add('fa-chevron-up');
    }
  }

  toggleButton.addEventListener('click', toggleSearchForm);
  setReload(true);            // also kicks off the initial contact fetch on mount

  return () => {
    toggleButton.removeEventListener('click', toggleSearchForm);   // cleanup
  };
});
```

Three things make this correct in React:

1. **It runs in an effect, not during render.** The DOM nodes only exist after React has committed the output, so `document.getElementById` would return `null` during render.
2. **No `DOMContentLoaded` wrapper.** That event fired long before this component mounted — see [Bug #5](#5-domcontentloaded-inside-a-react-effect).
3. **The returned function is the cleanup.** React calls it on unmount, detaching the listener so navigating away and back doesn't stack duplicate handlers.

The animation works by transitioning `max-height` from `0px` to the element's measured `scrollHeight` — a standard CSS trick, since `height: auto` is not animatable.

---

### Stage 5: Contact Detail

The detail page fetches **two** resources on mount: the contact itself, and its list of addresses.

**`src/lib/api/ContactApi.js`**

```javascript
export const contactDetail = async (token, id) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/contacts/${id}`, {
    method: 'GET',
    headers: { 'Content-Type': 'application/json', Authorization: token },
  });
};
```

**`src/components/Contact/ContactDetail.jsx`**

```jsx
export default function ContactDetail() {
  const { id } = useParams();                     // ← the :id segment from the URL
  const [token] = useLocalStorage('token', '');
  const [contact, setContact] = useState({});     // object — a single record
  const [address, setAddress] = useState([]);     // array — a list

  async function fetchContact() {
    const response = await contactDetail(token, id);
    const responseBody = await response.json();

    if (response.status === 200) {
      setContact(responseBody.data);
    } else {
      await alertError(responseBody.errors);
    }
  }

  async function fetchAddress() {
    const response = await addressList(token, id);
    const responseBody = await response.json();

    if (response.status === 200) {
      setAddress(responseBody.data);
    } else {
      await alertError(responseBody.errors);
    }
  }

  useEffectOnce(() => {
    fetchContact().then(() => console.log('Contact detail fetch successfully'));
    fetchAddress().then(() => console.log('Address fetch successfully'));
  });

  return (
    <>
      <p>{contact.first_name}</p>
      <p>{contact.last_name}</p>
      <p>{contact.email}</p>
      <p>{contact.phone}</p>

      {address.map((address) => (
        <div key={`address-${address.id}`}>
          <span>{address.street}</span>
          <span>{address.city}</span>
          {/* ...province, country, postal_code... */}
        </div>
      ))}
    </>
  );
}
```

- **`useParams()`** reads the dynamic segment declared in `main.jsx` (`<Route path={':id'}>`). Its value is always a **string** — the backend's Joi validation coerces it to a number, so no manual `parseInt` is needed here.
- **Initial state matters.** `contact` starts as `{}` so `contact.first_name` renders as empty rather than crashing during the first render (before the fetch resolves); `address` starts as `[]` so `.map()` is always safe to call.
- The two fetches are fired **in parallel** — neither awaits the other, since the address list only needs the contact's `id`, which already comes from the URL.

---

### Stage 6: Update Contact

**`src/lib/api/ContactApi.js`**

```javascript
export const contactUpdate = async (token, id, { first_name, last_name, email, phone }) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/contacts/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json', Authorization: token },
    body: JSON.stringify({ first_name, last_name, email, phone }),
  });
};
```

**`src/components/Contact/ContactEdit.jsx`**

```jsx
export default function ContactEdit() {
  const { id } = useParams();
  const [token] = useLocalStorage('token', '');

  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');
  const [phone, setPhone] = useState('');

  // 1) load the existing values into the form's state
  async function fetchContact() {
    const response = await contactDetail(token, id);
    const responseBody = await response.json();

    if (response.status === 200) {
      setFirstName(responseBody.data.first_name);   // ← .data, not the bare body
      setLastName(responseBody.data.last_name);
      setEmail(responseBody.data.email);
      setPhone(responseBody.data.phone);
    } else {
      await alertError(responseBody.errors);
    }
  }

  useEffectOnce(() => {
    fetchContact().then(() => console.log('Contact detail fetch successfully'));
  });

  // 2) submit whatever the state holds now
  async function handleSubmit(e) {
    e.preventDefault();

    const response = await contactUpdate(token, id, {
      first_name: firstName,
      last_name: lastName,
      email: email,
      phone: phone,
    });

    const responseBody = await response.json();

    if (response.status === 200) {
      await alertSuccess('Contact updated successfully');
    } else {
      await alertError(responseBody.errors);
    }
  }

  return (/* ...same four inputs as ContactCreate, but prefilled... */);
}
```

> **Key Insight:** An edit page is just a create page whose state is seeded by a fetch. Because the inputs are controlled, `setFirstName(...)` in the effect and `setFirstName(...)` in an `onChange` write to the same place — so "prefilled" and "user-edited" are indistinguishable by the time submit runs. This is also exactly where [Bug #2](#2-forgetting-the--data---response-envelope) showed up: read one level too shallow and the fetch succeeds while the form stays blank.

`PUT` here means a full replace: all four fields are always sent, even unchanged ones.

---

### Stage 7: Delete Contact

**`src/lib/api/ContactApi.js`**

```javascript
export const contactDelete = async (token, id) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/contacts/${id}`, {
    method: 'DELETE',
    headers: { 'Content-Type': 'application/json', Authorization: token },
  });
};
```

**`ContactList.jsx`**

```jsx
async function handleContactDelete(id) {
  if (await alertConfirm('Are you sure you want to delete this contact?')) {
    const response = await contactDelete(token, id);
    const responseBody = await response.json();

    if (response.status === 200) {
      alertSuccess(responseBody.message);
      setReload(true);            // refresh the list so the row disappears
    } else {
      alertError(responseBody.errors);
    }
  } else {
    return;                       // user cancelled — do nothing
  }
}
```

Two patterns worth keeping:

- **`await alertConfirm(...)` in the `if`** — the promise-based confirm reads like a synchronous question, with no callback nesting.
- **`setReload(true)` after a successful delete** — the UI reflects the server's new state by refetching rather than by mutating the local `contacts` array. Slightly more network traffic, but the pagination counts (`total_page`) stay correct, which local mutation would silently break.

---

## 📍 Part 3 — Address Management

### Address API Spec

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| POST | `/api/contacts/:contactId/addresses` | ✅ | Create a new address |
| GET | `/api/contacts/:contactId/addresses/:addressId` | ✅ | Get one address |
| PUT | `/api/contacts/:contactId/addresses/:addressId` | ✅ | Update an address |
| DELETE | `/api/contacts/:contactId/addresses/:addressId` | ✅ | Delete an address |
| GET | `/api/contacts/:contactId/addresses` | ✅ | List a contact's addresses |

Every address endpoint is nested under its contact — there is no standalone `/api/addresses`. The frontend routes mirror this exactly, so `useParams()` on an address page yields **both** ids:

```jsx
const { id, addressId } = useParams();   // from /dashboard/contacts/:id/addresses/:addressId/edit
```

---

### Stage 1: Create Address

**`src/lib/api/AddressApi.js`**

```javascript
export const addressCreate = async (
  token,
  id,
  { street, city, province, country, postal_code },
) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/contacts/${id}/addresses`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', Authorization: token },
    body: JSON.stringify({ street, city, province, country, postal_code }),
  });
};
```

**`src/components/Address/AddressCreate.jsx`**

```jsx
export default function AddressCreate() {
  const { id } = useParams();
  const [token] = useLocalStorage('token', '');
  const [contact, setContact] = useState('');      // parent contact, for the header
  const [street, setStreet] = useState('');
  const [city, setCity] = useState('');
  const [province, setProvince] = useState('');
  const [country, setCountry] = useState('');
  const [postalCode, setPostalCode] = useState('');
  const navigate = useNavigate();

  // fetch the parent contact purely to display "Adding an address for <name>"
  async function fetchContact() {
    const response = await contactDetail(token, id);
    const responseBody = await response.json();

    if (response.status === 200) {
      setContact(responseBody.data);
    } else {
      await alertError(responseBody.errors);
    }
  }

  useEffectOnce(() => {
    fetchContact().then(() => console.log('Contact detail fetched successfully'));
  });

  async function handleSubmit(e) {
    e.preventDefault();

    const response = await addressCreate(token, id, {
      street,
      city,
      province,
      country,
      postal_code: postalCode,     // camelCase state → snake_case field
    });
    const responseBody = await response.json();

    if (response.status === 200) {
      await alertSuccess('Address created successfully');
      navigate(`/dashboard/contacts/${id}`);       // back to the parent's detail page
    } else {
      await alertError(responseBody.errors);
    }
  }

  return (
    <>
      {/* context header so the user knows whose address they're adding */}
      <h2>{contact.first_name + ' ' + contact.last_name}</h2>
      <p>{contact.email + ' • ' + contact.phone}</p>

      <form onSubmit={handleSubmit}>{/* street, city, province, country, postal_code */}</form>
    </>
  );
}
```

> **Key Insight:** The extra `contactDetail` fetch here is purely for context — the create call itself only needs `id` from the URL. It's a small but real UX decision: an address form with no indication of *whose* address it is, is easy to fill in on the wrong contact.

---

### Stage 2: List Address

Addresses are listed inside `ContactDetail.jsx` (see [Part 2, Stage 5](#stage-5-contact-detail)), not on a page of their own.

**`src/lib/api/AddressApi.js`**

```javascript
export const addressList = async (token, id) => {
  return await fetch(`${import.meta.env.VITE_API_PATH}/api/contacts/${id}/addresses`, {
    method: 'GET',
    headers: { 'Content-Type': 'application/json', Authorization: token },
  });
};
```

Unlike contact search, this list has **no pagination** — the backend returns every address for the contact in one `{ data: [...] }` array.

---

### Stage 3: Update Address

**`src/lib/api/AddressApi.js`**

```javascript
export const addressDetail = async (token, id, addressId) => {
  return await fetch(
    `${import.meta.env.VITE_API_PATH}/api/contacts/${id}/addresses/${addressId}`,
    {
      method: 'GET',
      headers: { 'Content-Type': 'application/json', Authorization: token },
    },
  );
};

export const addressEdit = async (
  token,
  id,
  { addressId, street, city, province, country, postal_code },
) => {
  return await fetch(
    `${import.meta.env.VITE_API_PATH}/api/contacts/${id}/addresses/${addressId}`,
    {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json', Authorization: token },
      body: JSON.stringify({ street, city, province, country, postal_code }),
    },
  );
};
```

**`src/components/Address/AdressEdit.jsx`**

```jsx
export default function AddressEdit() {
  const { id, addressId } = useParams();        // both ids come from the URL
  const [token] = useLocalStorage('token', '');
  const [contact, setContact] = useState('');
  const [street, setStreet] = useState('');
  /* ...city, province, country, postalCode... */
  const navigate = useNavigate();

  async function fetchAddress() {
    const response = await addressDetail(token, id, addressId);
    const responseBody = await response.json();

    if (response.status === 200) {
      setStreet(responseBody.data.street);
      setCity(responseBody.data.city);
      setProvince(responseBody.data.province);
      setCountry(responseBody.data.country);
      setPostalCode(responseBody.data.postal_code);
    } else {
      await alertError(responseBody.errors);
    }
  }

  useEffectOnce(() => {
    fetchContact().then(() => console.log('Contact detail fetched successfully'));
    fetchAddress().then(() => console.log('Address detail fetched successfully'));
  });

  async function handleSubmit(e) {
    e.preventDefault();

    const response = await addressEdit(token, id, {
      addressId,                     // ← goes into the URL, not the body
      street, city, province, country,
      postal_code: postalCode,
    });
    const responseBody = await response.json();

    if (response.status === 200) {
      await alertSuccess('Address updated successfully');
      navigate(`/dashboard/contacts/${id}`);
    } else {
      await alertError(responseBody.errors);
    }
  }
}
```

Note the slightly different signature: `addressEdit` takes `addressId` **inside** the options object and lifts it out for the URL, while `addressDetail` and `addressDelete` take it as a positional argument. Both work, but the inconsistency is a small wart — worth standardizing if this were production code.

---

### Stage 4: Delete Address

**`src/lib/api/AddressApi.js`**

```javascript
export const addressDelete = async (token, contactId, addressId) => {
  return await fetch(
    `${import.meta.env.VITE_API_PATH}/api/contacts/${contactId}/addresses/${addressId}`,
    {
      method: 'DELETE',
      headers: { 'Content-Type': 'application/json', Authorization: token },
    },
  );
};
```

**`ContactDetail.jsx`**

```jsx
async function handleDelete(addressId) {
  if (await alertConfirm('Are you sure you want to delete this address?')) {
    const response = await addressDelete(token, id, addressId);
    const responseBody = await response.json();

    if (response.status === 200) {
      await fetchAddress();      // refetch the address list in place
    } else {
      await alertError(responseBody.errors);
    }
  } else {
    return;
  }
}
```

Same confirm-then-refetch pattern as contact delete — except here the refresh is a direct `await fetchAddress()` call rather than a `reload` flag, because this page has no pagination state that a stale closure could get wrong.

With that, all three modules are complete.

---

## 🛠️ Running the Project

**`package.json`** (scripts)

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "lint": "eslint .",
  "preview": "vite preview"
}
```

| Script | Command | When to use it |
| --- | --- | --- |
| `npm run dev` | `vite` | The everyday command. Starts the dev server with hot module replacement — saving a component updates the browser without a full reload or losing state. |
| `npm run build` | `vite build` | Produces the optimized production bundle in `dist/` — minified, tree-shaken, with hashed filenames. This is what gets deployed. |
| `npm run preview` | `vite preview` | Serves the built `dist/` folder locally. Use it to sanity-check the production build, since `dev` and `build` can differ (env handling, asset paths). |
| `npm run lint` | `eslint .` | Runs ESLint across the project — including `eslint-plugin-react-hooks`, which is where the rule in [Bug #8](#8-a-lint-rule-that-only-recognizes-reacts-own-effect-hooks) comes from. |

**Startup order matters:**

1. Start Postgres and make sure the database exists.
2. Start the backend: `node src/main.js` → `http://localhost:3000`.
3. Start the frontend: `npm run dev` → Vite prints the URL (default `http://localhost:5173`).

If the backend isn't running, every page still renders — the fetches simply fail, and SweetAlert shows the network error.

---

## 🧪 Manual Testing

There are no automated tests in this project, so verification is a walkthrough in the browser with DevTools open. The **Network** tab (to inspect the actual request headers and response bodies) and **Application → Local Storage** (to watch the token) are the two panels that matter.

**Happy path, in order:**

1. Open `/register` → create a user → expect a success alert and a redirect to `/login`.
2. Log in → check **Application → Local Storage** now holds a `token` key with a UUID value.
3. Land on `/dashboard/contacts` → the contact grid renders (empty except the "Create New Contact" card on a fresh account).
4. Create a contact → it appears in the list.
5. Open the contact → detail page shows its fields, and an empty address section.
6. Add an address → redirected back to the detail page, address card now visible.
7. Edit the contact and the address → confirm the forms are **prefilled** before you change anything.
8. Use the search panel (click the chevron to expand it) → filter by name/email/phone.
9. Create enough contacts to exceed one page (`size` defaults to 2) → click through the pagination buttons.
10. Delete an address, then a contact → confirm the SweetAlert dialog appears and the list refreshes after confirming.
11. Update your profile name and password from `/dashboard/users/profile`.
12. Log out → `token` disappears from Local Storage and you land on `/login`.

**Failure paths worth testing deliberately:**

| Test | Expected |
| --- | --- |
| Register with an existing username | `400` + error alert with the backend's message |
| Login with a wrong password | `401` + "Username or password wrong" |
| Manually corrupt the `token` in Local Storage, then reload a dashboard page | `401 Unauthorized` on the first fetch |
| Mismatched password + confirmation on register/profile | Blocked client-side, no request sent (check the Network tab) |
| Stop the backend, then submit a form | Network error surfaced through the alert |

> **Key Insight:** For every request the Network tab shows the **Request Headers** — this is the fastest way to confirm the `Authorization` header carries a bare UUID and not `Bearer <uuid>`, which is exactly what [Bug #3](#3-authorization-bearer-token-vs-a-raw-token-backend) turned on. Reading the response **Preview** also immediately reveals the `{ data: ... }` envelope that [Bug #2](#2-forgetting-the--data---response-envelope) was about.

---

## 🐛 Bugs Encountered & Fixed

Every bug below is real, hit while wiring this frontend to the backend. They're grouped by what they teach, not by when they happened.

### 1. Checking the wrong field for success

**Symptom:** Registration worked — the user row appeared in the database — but the UI always showed an error alert.

**Code:**

```javascript
const responseBody = await response.json();

// ❌
if (responseBody.statusCode === 200) {
  await alertSuccess('User registered successfully');
  navigate('/login');
} else {
  await alertError(responseBody.errors);
}
```

**Root cause:** The HTTP status code lives on the `Response` object (`response.status`), not inside the JSON body. The backend sends `{ "data": {...} }` — there is no `statusCode` field anywhere in it. So `responseBody.statusCode` was `undefined`, `undefined === 200` was always `false`, and the `else` branch ran on every single request, success or not.

**Fix:**

```javascript
// ✅
if (response.status === 200) { ... }
```

**Lesson:** `Response` and response *body* are two different objects. Status, headers, and `ok` come from the former; the payload comes from the latter.

---

### 2. Forgetting the `{ data: ... }` response envelope

**Symptom:** On the contact edit page, `console.log(responseBody)` clearly showed the contact — but every input rendered blank.

**Code:**

```javascript
// ❌
if (response.status === 200) {
  setFirstName(responseBody.first_name);
  setLastName(responseBody.last_name);
  setEmail(responseBody.email);
  setPhone(responseBody.phone);
}
```

**Root cause:** Every backend controller wraps a single resource in an envelope:

```javascript
res.status(200).json({ data: result });
```

So the payload is `{ data: { first_name: "...", ... } }`. Reading `responseBody.first_name` looks one level too shallow and yields `undefined` — and setting a controlled input's state to `undefined` renders an empty field, with no error.

**Fix:**

```javascript
// ✅
setFirstName(responseBody.data.first_name);
setLastName(responseBody.data.last_name);
setEmail(responseBody.data.email);
setPhone(responseBody.data.phone);
```

**Lesson:** `console.log` proving "the data is there" doesn't prove you're *reading it from the right path*. Log the exact expression you're about to use (`responseBody.data.first_name`), not just the whole object.

---

### 3. `Authorization: Bearer <token>` vs. a raw-token backend

**Symptom:** `GET /api/users/current` returned `401 Unauthorized` immediately after a successful login, with a token that was demonstrably valid and present in `localStorage`.

**Code:**

```javascript
// ❌ frontend
headers: {
  'Content-Type': 'application/json',
  Authorization: `Bearer ${token}`,
}
```

```javascript
// backend — auth-middleware.js
const token = req.get('Authorization');                             // "Bearer 3fa85f64-..."
const user = await prismaClient.user.findFirst({ where: { token } }); // looks for that whole string
```

**Root cause:** The backend never parses the `Bearer` scheme — it compares the header value **verbatim** against the `token` column, which stores a bare UUID (`user-service.js` generates it with `uuid()`). Searching for `"Bearer 3fa85f64-..."` matched no row, so the middleware returned 401. The token was fine; the *envelope around it* was not.

**Fix:**

```javascript
// ✅
headers: {
  'Content-Type': 'application/json',
  Authorization: token,
}
```

**Lesson:** `Authorization: Bearer <x>` is a convention, not a rule. Read the server's actual auth middleware before assuming the format — and when a 401 appears with a token you know is valid, compare the exact header string against the exact stored value.

---

### 4. A layout route with no `<Outlet />`

**Symptom:** Navigating to `/dashboard/users/profile` rendered the navbar and footer, but the profile content was missing. The route matched, the component mounted (its `console.log` fired), and the network request went out — yet nothing appeared.

**Code:**

```jsx
// ❌
export default function DashboardLayout() {
  return (
    <div>
      <header>{/* navbar */}</header>
      <main className="container mx-auto px-4 py-8 flex-grow">
        {/* nothing here — where should the child go? */}
        <div>© 2025 Contact Management.</div>
      </main>
    </div>
  );
}
```

**Root cause:** A layout route's `element` is only a shell. React Router has no way to guess where the matched child route should be rendered — that slot is declared explicitly with `<Outlet />`. Without it, the child component still mounts and runs its effects, but its output is never placed in the tree.

**Fix:**

```jsx
// ✅
import { Link, Outlet } from "react-router";

<main className="container mx-auto px-4 py-8 flex-grow">
  <Outlet />
  <div>© 2025 Contact Management.</div>
</main>
```

**Lesson:** "The component mounted but nothing rendered" in a nested-routing setup is almost always a missing `<Outlet />`. Note the misleading signal: side effects *do* run, so console logs and network calls make it look like rendering succeeded.

---

### 5. `DOMContentLoaded` inside a React effect

**Symptom:** The collapsible search panel's chevron button did nothing. No error in the console.

**Code:**

```javascript
// ❌
useEffectOnce(() => {
  document.addEventListener('DOMContentLoaded', function () {
    const toggleButton = document.getElementById('toggleSearchForm');
    /* ...setup... */
    toggleButton.addEventListener('click', toggleSearchForm);

    return () => {
      toggleButton.removeEventListener('click', toggleSearchForm);
    };
  });
});
```

**Root cause:** Two separate problems, both from copying static-template JavaScript verbatim:

1. `DOMContentLoaded` fires **once**, when the initial HTML document finishes parsing. By the time a React component mounts — especially one reached through client-side navigation — that event is long gone. A listener registered afterwards is never called, so the click handler was never attached.
2. The `return () => {...}` was returning from the `DOMContentLoaded` **callback**, not from the effect. Even if the outer problem were fixed, React would never receive that cleanup function.

**Fix:**

```javascript
// ✅
useEffectOnce(() => {
  const toggleButton = document.getElementById('toggleSearchForm');
  /* ...setup... */
  toggleButton.addEventListener('click', toggleSearchForm);

  return () => {
    toggleButton.removeEventListener('click', toggleSearchForm);   // now the effect's cleanup
  };
});
```

**Lesson:** An effect body *is* the "DOM is ready" callback — React runs it after committing to the DOM. When porting static HTML, `DOMContentLoaded` / `window.onload` wrappers should be deleted, not preserved. And always check which function a `return` actually belongs to when there's a nested callback.

---

### 6. One API function silently parsing the body itself

**Symptom:** The contact list stayed empty even though the API returned data. No visible error — the page just rendered zero contacts.

**Code:**

```javascript
// ❌ ContactApi.js — this one function breaks the convention
export const contactList = async (token, { name, phone, email, page }) => {
  const url = new URL(`${import.meta.env.VITE_API_PATH}/api/contacts`);
  /* ...searchParams... */

  const response = await fetch(url, { ... });
  return await response.json();      // ← returns a plain object, not a Response
};
```

```javascript
// ContactList.jsx — written against the convention every OTHER function follows
const response = await contactList(token, { name, phone, email, page });
const responseBody = await response.json();   // 💥 response.json is not a function
```

**Root cause:** `contactList` returned already-parsed JSON while every sibling function returned the raw `Response`. The caller then tried `.json()` on a plain object, throwing `TypeError: response.json is not a function`. Since `fetchContacts()` had no `try/catch` and the rejected promise was never awaited in a handler, the error surfaced only as an unhandled rejection in the console — `setContacts` was never reached, so the UI simply showed nothing. Even had it not thrown, `response.status` would have been `undefined` and the `=== 200` check would have failed.

**Fix:**

```javascript
// ✅ return the Response, like every other API function
export const contactList = async (token, { name, phone, email, page }) => {
  const url = new URL(`${import.meta.env.VITE_API_PATH}/api/contacts`);
  /* ...searchParams... */

  return await fetch(url, {
    method: 'GET',
    headers: { 'Content-Type': 'application/json', Authorization: token },
  });
};
```

**Lesson:** An internal API layer is a contract with itself. Pick one convention — raw `Response` **or** parsed body — and never mix, because callers are written against the pattern, not against each individual function. Also: a silently-empty UI often means an exception was swallowed; check the console for unhandled rejections before assuming the data is wrong.

---

### 7. Stale state read right after `setState`

**Symptom:** Pagination was consistently one click behind. Clicking page 2 highlighted "2" but still displayed page 1's contacts; clicking page 3 then showed page 2's.

**Code:**

```javascript
// ❌
async function handlePageChange(pageValue) {
  setPage(pageValue);        // schedules an update — `page` does NOT change here
  await fetchContacts();     // still closes over the OLD `page`
}

async function fetchContacts() {
  const response = await contactList(token, { name, phone, email, page });
  /* ... */
}
```

**Root cause:** `setState` in React is asynchronous. It schedules a re-render; it does not mutate the variable in the currently-executing function. `fetchContacts` reads `page` from the closure of the render that is still running — the previous value. Meanwhile the button highlight *did* update, because that's rendered from the new state after the re-render. Hence "the display and the data disagree by exactly one click."

**Fix — trigger the fetch from an effect instead:**

```javascript
// ✅
const [reload, setReload] = useState(false);

async function handlePageChange(pageValue) {
  setPage(pageValue);
  setReload(true);           // just ask for a refetch
}

useEffect(() => {
  if (reload) {
    fetchContacts();         // runs AFTER the re-render, so `page` is fresh
    setReload(false);
  }
}, [reload]);
```

**Lesson:** Never read a state variable on the line after setting it. If work depends on the *new* value, either pass that value explicitly as an argument, or run the work in an effect that fires after the render commits. The `reload`-flag pattern generalizes nicely: search, page change, and delete all funnel through one refetch path.

---

### 8. A lint rule that only recognizes React's own effect hooks

**Symptom:** After switching the contact fetch from `useEffectOnce` to `useEffect`, ESLint reported an error on code that had been silent — and still worked correctly:

```text
Error: Calling setState synchronously within an effect can trigger cascading renders
```

**Code:**

```jsx
// ⚠️ flagged
useEffect(() => {
  fetchContacts().then(() => console.log('Contacts fetched'));   // calls setContacts() inside
}, []);

// ✅ NOT flagged — functionally identical
useEffectOnce(() => {
  fetchContacts();
});
```

**Root cause:** `eslint-plugin-react-hooks` v7's `recommended` config enables a compiler-derived rule, `react-hooks/set-state-in-effect`, which statically traces whether a `setState` call is reachable from an effect body. Its detection only matches React's built-in hooks — `useEffect`, `useLayoutEffect`, `useInsertionEffect`. `react-use`'s `useEffectOnce` is just a wrapper the analysis doesn't recognize, so the identical fetch-on-mount pattern flew under the radar there.

**Fix (suppressing it for a legitimate fetch-on-mount):**

```jsx
useEffect(() => {
  // eslint-disable-next-line react-hooks/set-state-in-effect -- fetch-on-mount, not derived state
  fetchContacts().then(() => console.log('Contacts fetched'));
}, []);
```

Note the disable comment must sit on the **reported line** (the `fetchContacts()` call, which is where the diagnostic anchors), not on the `useEffect(` line — putting it a line too early leaves the error in place *and* adds an "unused eslint-disable directive" warning.

**Lesson:** The rule targets a real anti-pattern — deriving state in an effect when it could be computed during render — but fetching data on mount is a legitimate exception. Worth understanding *why* a rule fires before silencing it; and worth knowing that a lint rule's coverage can depend on which hook wrapper you happen to use.

---

## ⚠️ Known Remaining Issues

Things still present in the code, kept as future exercises:

| Issue | Where | Detail |
| --- | --- | --- |
| `alertSuccess(responseBody.message)` shows a blank dialog | `UserProfile.jsx`, `ContactList.jsx` | The backend never sends a `message` field — updates return `{ data: {...} }` and delete returns `{ data: "OK" }`. Should be a hardcoded string, like the other pages use. |
| No route guard on `/dashboard/*` | `main.jsx` | Any visitor can open a dashboard URL; they just get 401 alerts. A wrapper component that redirects to `/login` when `localStorage` has no token would be the fix. |
| `react-hooks/exhaustive-deps` warning | `ContactList.jsx` | `useEffect(..., [reload])` doesn't list `fetchContacts` as a dependency. Harmless here, but resolving it properly means `useCallback` or moving the function inside the effect. |
| Inconsistent `addressId` position | `AddressApi.js` | `addressEdit` takes it inside the options object; `addressDetail`/`addressDelete` take it positionally. |
| Filename typo | `AdressEdit.jsx` | Missing a `d` — "Adress" instead of "Address". |
| No loading or empty states | all list/detail pages | Nothing renders between mount and the fetch resolving, and an empty contact list looks identical to one that hasn't loaded yet. |

---

## 📚 Quick Reference

### All Endpoints

| Module | Method | Endpoint | Auth |
| --- | --- | --- | --- |
| User | POST | `/api/users` | - |
| User | POST | `/api/users/login` | - |
| User | GET | `/api/users/current` | ✅ |
| User | PATCH | `/api/users/current` | ✅ |
| User | DELETE | `/api/users/logout` | ✅ |
| Contact | POST | `/api/contacts` | ✅ |
| Contact | GET | `/api/contacts/:contactId` | ✅ |
| Contact | PUT | `/api/contacts/:contactId` | ✅ |
| Contact | DELETE | `/api/contacts/:contactId` | ✅ |
| Contact | GET | `/api/contacts` (search) | ✅ |
| Address | POST | `/api/contacts/:contactId/addresses` | ✅ |
| Address | GET | `/api/contacts/:contactId/addresses/:addressId` | ✅ |
| Address | PUT | `/api/contacts/:contactId/addresses/:addressId` | ✅ |
| Address | DELETE | `/api/contacts/:contactId/addresses/:addressId` | ✅ |
| Address | GET | `/api/contacts/:contactId/addresses` (list) | ✅ |

### Hooks Used

| Hook | Source | Used for |
| --- | --- | --- |
| `useState` | react | Every form field, fetched data, pagination, the `reload` flag |
| `useEffect` | react | The contact list's refetch-on-`reload` |
| `useEffectOnce` | react-use | Fetch-on-mount for detail/edit/profile pages, the DOM toggle setup |
| `useLocalStorage` | react-use | Reading/writing/clearing the session token |
| `useNavigate` | react-router | Programmatic redirect after an async result |
| `useParams` | react-router | Reading `:id` and `:addressId` from the URL |

### Component Responsibilities

| Component | Reads | Writes | Notes |
| --- | --- | --- | --- |
| `UserRegister` | — | `POST /api/users` | Client-side password confirmation |
| `UserLogin` | — | `POST /api/users/login` | Stores the token |
| `UserProfile` | `GET /api/users/current` | `PATCH /api/users/current` ×2 | Two independent forms |
| `UserLogout` | — | `DELETE /api/users/logout` | Renders nothing |
| `ContactList` | `GET /api/contacts` | `DELETE /api/contacts/:id` | Search + pagination + delete |
| `ContactCreate` | — | `POST /api/contacts` | |
| `ContactDetail` | `GET /api/contacts/:id` + address list | `DELETE .../addresses/:addressId` | Two parallel fetches |
| `ContactEdit` | `GET /api/contacts/:id` | `PUT /api/contacts/:id` | Prefilled form |
| `AddressCreate` | `GET /api/contacts/:id` (context) | `POST .../addresses` | |
| `AddressEdit` | contact + address detail | `PUT .../addresses/:addressId` | Two ids from `useParams` |

---

## 💡 Key Takeaways

- **The API layer is a contract, in two directions.** Every function in `lib/api/` must agree with its siblings (always return the raw `Response` — never mix in a pre-parsed one) *and* with the backend (the `{ data }` / `{ errors }` envelope, the raw-token `Authorization` header). Bugs #1, #2, #3, and #6 were all a broken contract, not broken UI logic.

- **`Response` ≠ response body.** Status and headers live on the `Response` object; the payload only exists after `await response.json()`. Conflating them is the single most common source of "it worked but the UI says it failed."

- **Nested routes need an explicit render slot.** `<Outlet />` isn't decorative — without it, child routes mount, run their effects, fire their requests, and render into nowhere. That combination of working side effects and invisible output makes it a confusing failure to diagnose.

- **`setState` is asynchronous — read the new value in the next render, not the next line.** The `reload`-flag + `useEffect` pattern is a small, reusable way to guarantee a fetch runs against fresh state, and it scales to any number of triggers (search, page change, delete) funnelling into one refetch path.

- **Effects are React's "DOM is ready" hook.** Code ported from static HTML carries assumptions that don't survive an SPA — `DOMContentLoaded` most obviously, but also `<a href>` navigation, which reloads the whole app and must become `<Link to>`.

- **The whole app needs no state management library.** The only truly global value is one token string, and `localStorage` already handles it. Everything else is either server state (refetched on demand) or local form state — a useful reminder that Redux is a solution to a problem this class of app doesn't have.

- **Editor tooling is configuration, not magic.** Autocomplete/auto-import works off `allowJs`, but the Quick Fix lightbulb needs an actual reported diagnostic, which needs `checkJs: true`. Knowing which feature depends on which setting saves a lot of "why doesn't my editor suggest this" time.
