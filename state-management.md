# 🗃️ State Management

Managing state effectively is crucial for optimizing your application's performance. Instead of storing all state information in a single centralized repository, consider dividing it into various categories based on their usage. By categorizing your state, you can streamline your state management process and enhance your application's overall efficiency.

## Component State

Component state is specific to individual components and should not be shared globally. It can be passed down to child components as props when necessary. Typically, you should begin by defining state within the component itself and consider elevating it to a higher level if it's required elsewhere in the application. When managing component state, you can use the following React hooks:

- [useState](https://react.dev/reference/react/useState) - for simpler states that are independent
- [useReducer](https://www.youtube.com/playlist?list=PLZlA0Gpn_vH8EtggFGERCwMY5u5hOjf-h) - for more complex states where on a single action you want to update several pieces of state

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  // const increment = () => setCount((prev) => prev + 1);
  const decrement = () => setCount(count + 1);
  // const decrement = () => setCount((prev) => prev - 1);

  return (
    <div>
      <h1>Counter: {count}</h1>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
};
```

Use **Functional update** `setCount(prev=>prev+1)` over **Direct update** `setCount(count+1)`

```jsx
// Example Direct Update
const Counter = () => {
  const [count, setCount] = useState(0);

  const incrementMultipleTimes = () => {
    for (let i = 0; i < 5; i++) {
      setCount(count + 1); // This will cause unexpected results
    }
  };

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={incrementMultipleTimes}>Increment 5 Times</button>
    </div>
  );
};

export default Counter;
```

Why using `setCount(count+1)` can cause unexpected results?

**State is Asynchronous:**

In the above code, when the button is clicked, `setCount(count + 1)` is called five times in quick succession inside the loop.

Since `setCount(count + 1)` is using the direct value of count (which is the state value at the time of the first call), all five setCount calls will be using the initial count value, which will not reflect the previous updates made by each setCount call.

Therefore, the final result will only reflect one update, not five.

To fix this, you should use the functional update form:

```jsx
for (let i = 0; i < 5; i++) {
  setCount((prev) => prev + 1);
}
```

Now, each `setCount` call will use the latest value of `count`, and the final result will be `5`, as expected.

## Application State

Unlike component state, which is limited to a single component, application state is used throughout the entire app.

For example, you might need global state for:

- A dark mode toggle that applies across multiple pages.
- A notification system that appears in different components.
- A user’s login status, required in the navbar, profile page, and settings.

Instead of making everything global from the beginning, only make the state global when needed. If a piece of state is only used in one component, keep it local for better performance.

When to Use Application State?

- Data needs to be shared across multiple components (e.g., user authentication, theme, notifications).
- You want to avoid prop drilling (passing data through multiple components manually).
- State must persist across different pages (e.g., user login info).

Good Application State Solutions:

- [context](https://react.dev/learn/passing-data-deeply-with-context) + [hooks](https://react.dev/reference/react-dom/hooks)
- [redux](https://redux.js.org/) + [redux toolkit](https://redux-toolkit.js.org/)
- [mobx](https://mobx.js.org)
- [zustand](https://github.com/pmndrs/zustand)
- [jotai](https://github.com/pmndrs/jotai)
- [xstate](https://xstate.js.org/)

```jsx
import { create } from "zustand";

export const useNotifications = create((set) => ({
  notifications: [],
  addNotification: (notification) => set((state) => [...state.notifications, { id: Date.now(), ...notification }]),
  dismissNotification: (id) => set((state) => state.notifications.filter((notification) => notification.id !== id)),
}));
```

## Server Cache State

The Server Cache State refers to the data retrieved from the server that is stored locally on the client-side for future use. While it is feasible to cache remote data within a state management store like Redux, there exist more optimal solutions to this practice. It is essential to consider more efficient caching mechanisms to enhance performance and optimize data retrieval processes.

Good Server Cache Libraries:

- [react-query](https://tanstack.com/query) - REST + GraphQL
- [swr](https://swr.vercel.app/) - REST + GraphQL
- [apollo client](https://www.apollographql.com/) - GraphQL
- [urql](https://formidable.com/open-source/urql/) - GraphQl
- [RTK](https://redux-toolkit.js.org/rtk-query)

```jsx
import { useQuery } from "@tanstack/react-query";
import api from "...";

const fetchUsers = () => api.get("/api/users").then((res) => res.data);

export const useUsers = () =>
  useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
    staleTime: 60000, // Data stays fresh for 1 min (no refetch)
    cacheTime: 300000, // Unused data stays cached for 5 min
    refetchOnWindowFocus: false, // No refetch when switching tabs
    refetchInterval: 10000, // Auto refetch every 10s
  });
```

## Form State

Forms are a crucial part of any application, and managing form state effectively is essential for a seamless user experience. When handling form state, consider using libraries like React Hook Form to streamline the process. These libraries provide built-in validation, error handling, and form submission functionalities, making it easier to manage form state within your application

Forms in React can be [controlled and uncontrolled](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components).

Although it is possible to build any form using only React primitives, there are some good solutions out there that help with handling forms such as:

- [React Hook Form](https://react-hook-form.com/)
- [Formik](https://formik.org/)
- [React Final Form](https://github.com/final-form/react-final-form)

You can also integrate validation libraries with the mentioned solutions to validate inputs on the client. Some good options are:

- [zod](https://github.com/colinhacks/zod)
- [yup](https://github.com/jquense/yup)

[Shadcn UI Form Code Example](https://ui.shadcn.com/docs/components/form)

## URL State

URL state refers to the data stored and manipulated within the address bar of the browser. This state is commonly managed through URL parameters `(e.g., /app/${dynamicParam})` or query parameters `(e.g., /app?dynamicParam=1)`. By incorporating routing solutions like react-router

```jsx
import { useParams } from "react-router/dom";

const DiscussionPage = () => {
  const { discussionId } = useParams(); // Extract discussionId from URL

  return <DiscussionView discussionId={discussionId} />;
};
```
