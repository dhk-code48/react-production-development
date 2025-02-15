# 🚄 Performance

## Code Splitting

Code splitting is the process of breaking production JavaScript into smaller chunks to improve application load times. This approach allows the app to download and execute only the necessary code when required, rather than loading everything at once.

For optimal performance, code splitting should primarily be applied at the route level, ensuring that only essential code is loaded initially while additional parts are fetched lazily when needed. However, excessive code splitting can negatively impact performance by increasing the number of requests needed to retrieve all code chunks. A well-planned strategy that focuses on splitting critical parts of the application ensures a balance between performance optimization and efficient resource loading.

<video src="./assets/code-splitting.mp4" controls>
  Your browser does not support the video tag.
</video>

```jsx
// Without Lazy Loading
import { Routes, Route } from "react-router-dom";
import Home from "..";
import About from "..";
import Contact from "..";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  );
}

// With Lazy Loading
import { lazy } from "react";

const Home = lazy(() => import(".."));
const About = lazy(() => import(".."));
const Contact = lazy(() => import(".."));

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  );
}

export default App;
```

<details>
<summary><strong>More Advance Example</strong></summary>

```jsx
import { QueryClient, useQueryClient } from "@tanstack/react-query";
import { useMemo } from "react";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import { ProtectedRoute } from "@/lib/auth";

// Lazy loader helper (similar to <Suspense> </Suspense>)
const convert = (queryClient: QueryClient) => (module: any) => ({
  Component: module.default,
  loader: module.loader?.(queryClient),
});

export const createAppRouter = (queryClient: QueryClient) =>
  createBrowserRouter([
    {
      path: "/",
      lazy: () => import("./Home").then(convert(queryClient)),
    },
    {
      path: "/login",
      lazy: () => import("./Login").then(convert(queryClient)),
    },
    {
      path: "/dashboard",
      element: (
        <ProtectedRoute>
          <div>Dashboard</div>
        </ProtectedRoute>
      ),
    },
    {
      path: "*",
      lazy: () => import("./NotFound").then(convert(queryClient)),
    },
  ]);

// Router component
export const AppRouter = () => {
  const queryClient = useQueryClient();
  const router = useMemo(() => createAppRouter(queryClient), [queryClient]);

  return <RouterProvider router={router} />;
};
```

</details>

## Component and state optimizations

- Do not put everything in a single state. That might trigger unnecessary re-renders. Instead split the global state into multiple states according to where they are being used.
- Use the state initializer function to ensure an expensive computation runs only once, instead of running it on every re-render. For example:-

```jsx
// instead of this which would be executed on every re-render:
const [state, setState] = React.useState(myExpensiveFn());

// prefer this which is executed only once:
const [state, setState] = React.useState(() => myExpensiveFn());
```

- Important to remember, context(global state) is often used as the "golden tool" for props drilling, whereas in many scenarios you may satisfy your needs by [`lifting the state up`](https://react.dev/learn/sharing-state-between-components#lifting-state-up-by-example) or a [`proper composition of components`](https://react.dev/learn/passing-data-deeply-with-context#before-you-use-context). Do not rush with context and global state.

## Children as the most basic optimization

The `children` prop is the most basic and easiest way to optimize your components. When applied properly, it eliminates a lot of unnecessary rerenders. The JSX, passed in the form of children prop, represents an isolated VDOM structure that does not need (and cannot) be re-rendered by its parent. Example below:

```jsx
// Not optimized example
const App = () => <Counter />;

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount((count) => count + 1)}>count is {count}</button>
      <PureComponent /> // will rerender whenever "count" updates
    </div>
  );
};

const PureComponent = () => <p>Pure Component</p>;

// Optimized example
const App = () => (
  <Counter>
    <PureComponent />
  </Counter>
);

const Counter = ({ children }) => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount((count) => count + 1)}>count is {count}</button>
      {children} // won't rerender whenever "count" updates
    </div>
  );
};

const PureComponent = () => <p>Pure Component</p>;
```

## Image optimizations

- Consider lazy loading images that are not in the viewport.

- Use modern image formats such as WEBP for faster image loading.

- Use `srcset` to load the most optimal image for the clients screen size.

## Web vitals

Since Google started taking web vitals in account when indexing websites, you should keep an eye on web vitals scores from [Lighthouse](https://pagespeed.web.dev/) and [Pagespeed Insights](https://pagespeed.web.dev/).

## Data prefetching

It is possible to prefetch data before the user navigates to a page. This can be done by using the queryClient.prefetchQuery method from the @tanstack/react-query library. This method allows you to prefetch data for a specific query. This can be useful when you know that the user will navigate to a specific page and you want to prefetch the data before the user navigates to the page. This can help to improve the performance of the application by reducing the time it takes to load the data when the user navigates to the page.

```jsx
<Link
  onMouseEnter={() => {
    // Prefetch the data when the user hovers over the link
    queryClient.prefetchQuery(getNecessaryData(id));
  }}
  to="..."
>
  View
</Link>
```
