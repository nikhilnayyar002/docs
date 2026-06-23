7.16.0
https://reactrouter.com

- [1. Getting Started](#1-getting-started)
  - [1.1. Picking a Mode](#11-picking-a-mode)
  - [1.2. Declarative Mode](#12-declarative-mode)
    - [1.2.1. Installation](#121-installation)
    - [1.2.2. Routing](#122-routing)
    - [1.2.3. Navigating](#123-navigating)
    - [1.2.4. URL Values](#124-url-values)
- [2. How-Tos](#2-how-tos)
  - [2.1. Accessibility](#21-accessibility)
- [3. Explanations](#3-explanations)
  - [3.1. React Transitions](#31-react-transitions)

# 1. Getting Started

## 1.1. Picking a Mode

There are three primary ways, or "modes", to use it in your app. The features available in each mode are additive, so moving from Declarative to Data to Framework simply adds more features.

## 1.2. Declarative Mode

### 1.2.1. Installation

`npm i react-router`

```tsx
const root = document.getElementById("root");
ReactDOM.createRoot(root).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
);
```

### 1.2.2. Routing

```tsx
<Routes>
  <Route index element={<Home />} />
  <Route path="about" element={<About />} />

  <Route element={<AuthLayout />}>
    <Route path="login" element={<Login />} />
  </Route>

  <Route path="concerts">
    <Route index element={<ConcertsHome />} />
  </Route>
</Routes>
```

Child routes are rendered through the `<Outlet/>` in the parent route.

```tsx
// dynamic segment
<Route path="/c/:categoryId/p/:productId" element={<Product />}/>
let { categoryId, productId } = useParams();

// Optional Segments
<Route path=":lang?/categories" element={<Categories />} />
<Route path="users/:userId/edit?" element={<User />} />

// catch all 
<Route path="files/*" element={<File />} />

// Linking
<NavLink to="/" className={({ isActive }) => ...}>Home</NavLink>
<Link to="/concerts/salt-lake-city">Concerts</Link>
```

### 1.2.3. Navigating

Whenever a NavLink is active, it will automatically have an style from `a.active` selector. It also has callback props on `className`, `style`, and `children`.

```tsx
let navigate = useNavigate();
navigate("/dashboard");
```

### 1.2.4. URL Values

```tsx
let { city } = useParams(); // /concerts/:city
let [searchParams] = useSearchParams(); // values after a ? in the URL
searchParams.get("q")
let location = useLocation();
location.pathname
```

# 2. How-Tos

## 2.1. Accessibility

The `<Link>` component renders a standard anchor tag.

`<NavLink/>` which behaves the same as `<Link>`, but it also provides context for assistive technology when the link points to the current page. This is useful for building navigation menus or breadcrumbs.

# 3. Explanations

## 3.1. [React Transitions](https://reactrouter.com/explanation/react-transitions#transitions-in-react-router)

