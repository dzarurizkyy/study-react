# ⚛️ React.js Study Case — Contact Management

Study notes for building the **Contact Management** frontend in React — a Vite + React Router SPA that consumes an existing Node.js/Express/Prisma REST API. No global state library (Redux, Zustand, etc.) is used: all data lives on the backend, the session token lives in `localStorage`, and the UI is just React + React Router + a couple of small hook/alert utilities. This note walks through the setup, the routing/API architecture, each feature module (User → Contact → Address), and — the most useful part — every real integration bug hit while wiring the frontend to the backend, with the root cause and the fix.

---

## 📋 Table of Contents

- [Requirement](#-requirement)
- [Backend Setup](#-backend-setup)
- [Frontend Setup](#-frontend-setup)
- [Project Structure](#-project-structure)
- [Routing](#-routing)
- [API Layer & Authentication](#-api-layer--authentication)
- [Part 1 — User Management](#-part-1--user-management)
- [Part 2 — Contact Management](#-part-2--contact-management)
- [Part 3 — Address Management](#-part-3--address-management)
- [Running the Project](#-running-the-project)
- [Bugs Encountered & Fixed](#-bugs-encountered--fixed)
- [Quick Reference](#-quick-reference)
- [Key Takeaways](#-key-takeaways)

---

## 🎯 Requirement

Build the frontend for: [web-template-contact-management](https://github.com/ProgrammerZamanNow/web-template-contact-management)

The app has 3 features, matching 3 backend modules:

1. **User Management** — register, login, view/update profile, logout
2. **Contact Management** — contact CRUD + search & pagination
3. **Address Management** — address CRUD, an address always belongs to a contact

> **Key Insight:** This class deliberately reuses an already-built backend (Node.js/Express/Prisma) instead of writing a new one, specifically so the same language (JavaScript) is used on both ends while focusing entirely on the frontend/SPA concerns: routing, forms, client-side auth state, and talking to a REST API.

---

## 🏗️ Backend Setup

The backend is a separate, pre-existing project: [belajar-nodejs-restful-api](https://github.com/ProgrammerZamanNow/belajar-nodejs-restful-api).

```bash
git clone git@github.com:ProgrammerZamanNow/belajar-nodejs-restful-api.git
cd belajar-nodejs-restful-api
npm install
npx prisma generate
```

**Database**

```sql
create database belajar_reactjs_contact_management;
```

```bash
# configure DATABASE_URL in .env, then:
npx prisma migrate dev
```

**CORS**

The backend and frontend run on different ports during local development, so the browser blocks cross-origin requests by default. The backend needs `cors` enabled:

```bash
npm install cors
```

**Run it**

```bash
node src/main.js
# Backend listens on http://localhost:3000
```

---

## ⚛️ Frontend Setup

```bash
npm create vite@latest study-reatjs-contact-management -- --template react
npm install react-router
```

**React Use** — a popular hooks utility library (`useEffectOnce`, `useLocalStorage`, etc.), used throughout this project instead of hand-rolling the same patterns.

```bash
npm install react-use
```

> Reference: [github.com/streamich/react-use](https://github.com/streamich/react-use)

**SweetAlert2** — nicer alert/confirm dialogs than the browser's built-in `alert()`/`confirm()`.

```bash
npm install sweetalert2
```

> Reference: [sweetalert2.github.io](https://sweetalert2.github.io/)

**Environment**

```
# .env
VITE_API_PATH=http://localhost:3000
```

Every API call reads the backend base URL from `import.meta.env.VITE_API_PATH`, so switching environments (local, staging) never requires touching the API layer's code.

---

## 🗂️ Project Structure

```
src/
├── components/
│   ├── Layout.jsx            # public layout (register/login) — renders <Outlet />
│   ├── DashboardLayout.jsx   # authenticated layout (navbar) — renders <Outlet />
│   ├── User/
│   │   ├── UserRegister.jsx
│   │   ├── UserLogin.jsx
│   │   ├── UserProfile.jsx
│   │   └── UserLogout.jsx
│   ├── Contact/
│   │   ├── ContactList.jsx   # search + pagination + delete
│   │   ├── ContactCreate.jsx
│   │   ├── ContactDetail.jsx # contact info + its addresses
│   │   └── ContactEdit.jsx
│   └── Address/
│       ├── AddressCreate.jsx
│       └── AdressEdit.jsx
├── lib/
│   ├── alert.js               # sweetalert2 wrappers
│   └── api/
│       ├── UserApi.js
│       ├── ContactApi.js
│       └── AddressApi.js
└── main.jsx                   # routes
```

> **Key Insight:** No `pages/` vs `components/` split, no state management folder — because routing is flat (React Router) and state is either local (`useState`) or just the session token (`localStorage`), the whole app fits in `components/` + `lib/`.

---

## 🧭 Routing

**`src/main.jsx`**

```jsx
<BrowserRouter>
  <Routes>
    <Route element={<Layout />}>
      <Route path="register" element={<UserRegister />} />
      <Route path="login" element={<UserLogin />} />
    </Route>

    <Route path="dashboard" element={<DashboardLayout />}>
      <Route path="users">
        <Route path="profile" element={<UserProfile />} />
        <Route path="logout" element={<UserLogout />} />
      </Route>
      <Route path="contacts">
        <Route index element={<ContactList />} />
        <Route path="create" element={<ContactCreate />} />
        <Route path=":id">
          <Route index element={<ContactDetail />} />
          <Route path="edit" element={<ContactEdit />} />
          <Route path="addresses">
            <Route path="create" element={<AddressCreate />} />
            <Route path=":addressId/edit" element={<AddressEdit />} />
          </Route>
        </Route>
      </Route>
    </Route>
  </Routes>
</BrowserRouter>
```

Two layout routes wrap everything:

```jsx
// Layout.jsx — public pages (register/login)
export default function Layout() {
  return (
    <div className="...">
      <Outlet />
    </div>
  );
}
```

```jsx
// DashboardLayout.jsx — authenticated pages
export default function DashboardLayout() {
  return (
    <div className="...">
      <header>{/* navbar */}</header>
      <main>
        <Outlet />
        <footer>...</footer>
      </main>
    </div>
  );
}
```

> **Key Insight:** A layout route's `element` is only the *shell*. React Router only knows where to inject the matched child route (`ContactList`, `UserProfile`, ...) because of `<Outlet />`. Forget it, and the layout renders fine but every child page silently renders nothing — the route matched, the component mounted, but there's no slot for it to appear in.

---

## 🔌 API Layer & Authentication

Every endpoint call lives in `src/lib/api/*.js`, one file per resource, and follows the same shape: build the URL from `VITE_API_PATH`, `fetch`, and **return the raw `Response`** — parsing (`.json()`) and status checks are left to the calling component.

**`src/lib/api/UserApi.js`**

```js
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
```

**Auth**: the backend represents a session as a plain `token` string (a `uuid`, generated on login and stored on the `users` row). The frontend keeps it in `localStorage` via `react-use`'s `useLocalStorage('token', '')`, and sends it back as-is in the `Authorization` header — **no `Bearer ` prefix** (see [Bug #3](#-bugs-encountered--fixed) below for why that matters).

```js
const [token, setToken] = useLocalStorage('token', '');
```

**`src/lib/alert.js`** — thin sweetalert2 wrappers used everywhere instead of calling `Swal.fire` directly:

```js
export const alertSuccess = async (message) =>
  Swal.fire({ icon: 'success', title: 'Success', text: message });

export const alertError = async (message) =>
  Swal.fire({ icon: 'error', title: 'Error', text: message });

export const alertConfirm = async (message) => {
  const result = await Swal.fire({
    icon: 'question',
    title: 'Are you sure?',
    text: message,
    showCancelButton: true,
    confirmButtonText: 'Yes',
  });
  return result.isConfirmed; // usable directly in an `if`
};
```

---

## 👤 Part 1 — User Management

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/users` | - | Register a new user |
| POST | `/api/users/login` | - | Login, receive a token |
| GET | `/api/users/current` | ✅ | Get the logged-in user's data |
| PATCH | `/api/users/current` | ✅ | Update name/password |
| DELETE | `/api/users/logout` | ✅ | Clear the token (logout) |

Every success response is wrapped as `{ "data": { ... } }`; every failure as `{ "errors": "..." }`.

**Register → Login → Profile → Logout**, in code:

```jsx
// UserRegister.jsx
const response = await userRegister({ username, password, name });
const responseBody = await response.json();

if (response.status === 200) {
  await alertSuccess('User registered successfully');
  navigate('/login');
} else {
  await alertError(responseBody.errors);
}
```

```jsx
// UserLogin.jsx — stash the token on success
const response = await userLogin({ username, password });
const responseBody = await response.json();

if (response.status === 200) {
  setToken(responseBody.data.token);
  await alertSuccess('Login success');
  navigate('/dashboard/contacts');
} else {
  await alertError(responseBody.errors);
}
```

```jsx
// UserProfile.jsx — fetch once on mount
async function fetchUserDetail() {
  const response = await userDetail(token);
  const responseBody = await response.json();

  if (response.status === 200) {
    setName(responseBody.data.name);
  } else {
    alertError(responseBody.errors);
  }
}

useEffectOnce(() => {
  fetchUserDetail();
});
```

```jsx
// UserLogout.jsx — no UI, just fires the logout call and redirects
useEffectOnce(() => {
  handleLogout(); // clears local token, calls the API, navigate('/login')
});
```

---

## 📇 Part 2 — Contact Management

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/contacts` | ✅ | Create a new contact |
| GET | `/api/contacts/:contactId` | ✅ | Get one contact's detail |
| PUT | `/api/contacts/:contactId` | ✅ | Update a contact |
| DELETE | `/api/contacts/:contactId` | ✅ | Delete a contact |
| GET | `/api/contacts?name=&email=&phone=&page=&size=` | ✅ | Search + pagination |

`ContactList.jsx` is the most involved page — search, pagination, and delete all live here:

```jsx
async function fetchContacts() {
  const response = await contactList(token, { name, phone, email, page });
  const responseBody = await response.json();
  if (response.status === 200) {
    setContacts(responseBody.data);
    setTotalPages(responseBody.paging.total_page);
  } else {
    await alertError(responseBody.errors);
  }
}

// Every user action (search submit, page click, delete) just flips
// `reload` — the effect below is the single place fetchContacts() runs from.
useEffect(() => {
  if (reload) {
    fetchContacts();
    setReload(false);
  }
}, [reload]);

async function handlePageChange(pageValue) {
  setPage(pageValue);
  setReload(true);
}

async function handleContactDelete(id) {
  if (await alertConfirm('Are you sure you want to delete this contact?')) {
    const response = await contactDelete(token, id);
    if (response.status === 200) {
      setReload(true);
    }
  }
}
```

> **Key Insight:** Routing every mutation (search, page change, delete) through one `reload` flag + one `useEffect` guarantees `fetchContacts()` always runs *after* React has committed the latest `page`/`name`/`email`/`phone` state — see [Bug #7](#-bugs-encountered--fixed) for what goes wrong without it.

`ContactCreate.jsx` and `ContactEdit.jsx` are a standard controlled-form pair (`useState` per field, `onSubmit` calls the API, `alertSuccess`/`alertError` on the result). `ContactDetail.jsx` fetches both the contact **and** its address list on mount, and renders a delete button per address via `alertConfirm`.

---

## 📍 Part 3 — Address Management

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/contacts/:contactId/addresses` | ✅ | Create a new address |
| GET | `/api/contacts/:contactId/addresses/:addressId` | ✅ | Get one address |
| PUT | `/api/contacts/:contactId/addresses/:addressId` | ✅ | Update an address |
| DELETE | `/api/contacts/:contactId/addresses/:addressId` | ✅ | Delete an address |
| GET | `/api/contacts/:contactId/addresses` | ✅ | List a contact's addresses |

Addresses are always scoped under a `contactId` in the URL — there's no top-level `/api/addresses`. `AddressCreate.jsx` and `AdressEdit.jsx` both fetch the parent contact first (to show "Add address for John Doe" context), then submit the address form:

```jsx
// AdressEdit.jsx
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
```

---

## 🛠️ Running the Project

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "lint": "eslint .",
  "preview": "vite preview"
}
```

1. Start the backend first: `node src/main.js` (port `3000`).
2. Start the frontend: `npm run dev` (Vite's default port).
3. Register a user, log in, then create/search/edit contacts and addresses.

---

## 🐛 Bugs Encountered & Fixed

These are real bugs hit while wiring this frontend to the backend — each one is a small, generally-applicable lesson, not just a typo.

### 1. Checking the wrong field for success

```js
// ❌ responseBody never has a `statusCode` field — the HTTP status
// lives on the Response object, not inside the JSON body.
if (responseBody.statusCode === 200) { ... }

// ✅
if (response.status === 200) { ... }
```
The condition was always `false`, so the app kept showing an error alert even when the user was successfully registered.

### 2. Forgetting the `{ data: ... }` response envelope

Every backend controller wraps a single-resource response as `{ "data": {...} }` — never the bare object.

```js
// ❌ responseBody.first_name is undefined
setFirstName(responseBody.first_name);

// ✅
setFirstName(responseBody.data.first_name);
```
The fetch succeeded (confirmed via `console.log`), but the form fields stayed empty because the fields were read one level too shallow.

### 3. `Authorization: Bearer <token>` vs. a raw-token backend

```js
// backend's auth-middleware.js compares the header value directly,
// with no scheme prefix:
const token = req.get('Authorization');
const user = await prismaClient.user.findFirst({ where: { token } });
```

```js
// ❌ backend looks for a row with token === "Bearer 3fa85f64-...", finds nothing → 401
Authorization: `Bearer ${token}`,

// ✅
Authorization: token,
```
A perfectly valid token still failed authentication because of a header-format mismatch between frontend and backend — always read *both* sides of an integration, not just one.

### 4. A layout route with no `<Outlet />`

```jsx
// ❌ DashboardLayout renders header/nav/footer only — no slot for children
export default function DashboardLayout() {
  return (
    <div>
      <header>...</header>
      <main>{/* nothing here */}</main>
    </div>
  );
}

// ✅
<main>
  <Outlet />
</main>
```
Routing matched correctly, the child component (`UserProfile`, `ContactList`, ...) even mounted — it just had nowhere to render.

### 5. `DOMContentLoaded` inside a React effect

Leftover from the original static-HTML template:

```js
// ❌ DOMContentLoaded already fired before this component ever mounted —
// this callback never runs, and the search-form toggle button is dead.
useEffectOnce(() => {
  document.addEventListener('DOMContentLoaded', () => {
    toggleButton.addEventListener('click', toggleSearchForm);
  });
});

// ✅ the DOM is already ready by the time an effect runs — just wire it up directly
useEffectOnce(() => {
  toggleButton.addEventListener('click', toggleSearchForm);
  return () => toggleButton.removeEventListener('click', toggleSearchForm);
});
```
`DOMContentLoaded` fires once, for the initial page load — in an SPA, a component that mounts later (e.g. after client-side navigation) has already missed it.

### 6. One API function silently parsing the body itself

```js
// ❌ contactList parses the JSON internally...
export const contactList = async (token, params) => {
  const response = await fetch(url, { ... });
  return await response.json(); // now a plain object, not a Response
};

// ...but the caller expects a Response, like every other API function:
const response = await contactList(token, params);
await response.json(); // 💥 TypeError: response.json is not a function
```
Every other function in the same file returns the raw `fetch()` `Response`. One inconsistent function broke the calling component's assumption and crashed silently (no try/catch around it) before `setContacts` ever ran.

### 7. Stale state read right after `setState`

```js
// ❌ setPage schedules the update — `page` inside fetchContacts()
// still closes over the OLD value when called on the very next line.
async function handlePageChange(pageValue) {
  setPage(pageValue);
  await fetchContacts(); // fetches the previous page, one click behind
}
```
```js
// ✅ let the state update land first, then react to it in an effect
async function handlePageChange(pageValue) {
  setPage(pageValue);
  setReload(true);
}

useEffect(() => {
  if (reload) {
    fetchContacts(); // now closes over the fresh `page`
    setReload(false);
  }
}, [reload]);
```
`setState` in React is asynchronous — code right after it still sees the value from the render that just finished, not the one being scheduled.

### 8. A new ESLint rule that only recognizes React's own effect hooks

`eslint-plugin-react-hooks`'s `recommended` config (v7+) ships a compiler-derived rule, `react-hooks/set-state-in-effect`, that flags `setState` reachable from an effect body:

```jsx
// this DID get flagged:
useEffect(() => {
  fetchContacts().then(() => {}); // fetchContacts() calls setContacts() internally
}, []);

// this did NOT get flagged, even though it does the exact same thing:
useEffectOnce(() => {
  fetchContacts();
});
```
The rule's static analysis only recognizes `useEffect` / `useLayoutEffect` / `useInsertionEffect` — not `react-use`'s `useEffectOnce`. Switching from one to the other is what made a previously-silent warning suddenly appear on an unchanged, legitimate fetch-on-mount pattern.

---

## 📚 Quick Reference

### All Endpoints

| Module | Method | Endpoint | Auth |
|---|---|---|---|
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

---

## 💡 Key Takeaways

- **A frontend API layer is a contract, not just plumbing.** Every function in `lib/api/*.js` needs to agree with itself (always return `Response`, or always return parsed JSON — never mix) *and* with the backend it's calling (envelope shape, header format). Most of the bugs above were a mismatch on one side of that contract, not a typo in the UI code.
- **Nested routes need an explicit render target.** `<Outlet />` is not optional boilerplate — without it a layout route silently swallows every child page.
- **Code ported from static HTML carries assumptions that don't hold in an SPA** — `DOMContentLoaded` being the clearest example. Anything that depended on the browser's page-load lifecycle needs to be re-anchored to React's component lifecycle instead.
- **`setState` is asynchronous — read the *next* render's state in an effect, not the next line.** A `reload`-flag + `useEffect` pattern is a simple, reusable way to guarantee a fetch runs against fresh state.
- **Auth here is a raw token string in a header, not JWT.** Simple to learn with, but it means the exact header format (`Authorization: <token>`, no scheme prefix) is part of the contract and has to match the backend's `authMiddleware` exactly.
