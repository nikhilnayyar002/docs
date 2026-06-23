v19.2

- [1. react@19.2](#1-react192)
  - [1.1. hooks](#11-hooks)
    - [1.1.1. useContext \*](#111-usecontext-)
    - [1.1.2. useTransition \*](#112-usetransition-)
    - [1.1.3. useActionState \*](#113-useactionstate-)
    - [1.1.4. useOptimistic \*](#114-useoptimistic-)
    - [1.1.5. useDebugValue](#115-usedebugvalue)
    - [1.1.6. useDeferredValue \*](#116-usedeferredvalue-)
    - [useEffect \*](#useeffect-)
    - [1.1.7. useEffectEvent \*](#117-useeffectevent-)
    - [1.1.8. useId](#118-useid)
    - [1.1.9. useImperativeHandle](#119-useimperativehandle)
    - [1.1.10. useLayoutEffect](#1110-uselayouteffect)
    - [1.1.11. useSyncExternalStore](#1111-usesyncexternalstore)
  - [1.2. Components](#12-components)
    - [1.2.1. `<Suspense>`](#121-suspense)
    - [1.2.2. `<Activity>`](#122-activity)
  - [1.3. APIs](#13-apis)
    - [1.3.1. use](#131-use)
- [2. react-dom@19.2](#2-react-dom192)
  - [2.1. Hooks](#21-hooks)
    - [2.1.1. useFormStatus](#211-useformstatus)
  - [2.2. Components](#22-components)
    - [2.2.1. `<form>`](#221-form)
    - [2.2.2. `<input>`](#222-input)
  - [2.3. APIs](#23-apis)
    - [2.3.1. createPortal, flushSync,](#231-createportal-flushsync)
    - [2.3.2. preconnect, prefetchDNS,](#232-preconnect-prefetchdns)
    - [2.3.3. preinit, preinitModule](#233-preinit-preinitmodule)
    - [2.3.4. preloadModule, preload](#234-preloadmodule-preload)
- [3. Rules of React](#3-rules-of-react)

# 1. react@19.2

## 1.1. hooks

### 1.1.1. useContext *

- https://react.dev/reference/react/useContext#overriding-context-for-a-part-of-the-tree
  
  `useContext()` always looks for the closest provider above the component that calls it

### 1.1.2. useTransition *

lets you render a part of the UI in the background.

Functions called in `startTransition` are called “Actions”. By convention, any callback called inside `startTransition` (such as a callback prop) should be named action or include the “Action” suffix.

React calls the function (**action**) passed to `startTransition` immediately and marks all state updates scheduled synchronously during the action function call as Transitions.
Any async calls that are awaited in the `action` will be included in the Transition, but currently require wrapping any `set` functions after the await in an additional `startTransition`.

You can wrap an update into a Transition only if you have access to the set function of that state. If you want to start a Transition in response to some prop or a custom Hook value, try `useDeferredValue` instead.

A state update marked as a Transition is non-blocking and will be interrupted by blocking state updates (urgent) like user input. React will restart the rendering work (transition) after doing the urgent rendering.

If there are multiple ongoing Transitions, React currently batches them together.
> - React intercepts all startTransition calls executed within the same synchronous event handler or macro-task loop.
> - If Transition A (the analytics data) suspends because it needs to fetch a heavy chart layout, the entire virtual branch stays hidden in the background. React will not render the sidebar closing . It waits until every single piece of data required by the batched transitions is completely resolved.
> - You can separate the updates into different browser execution ticks using `setTimeout`
>   ```
>   startTransition(() => ...); 
>   setTimeout(() => { startTransition(() => setActiveTab('analytics')); }, 0);
>   ```
>   Transitions are designed for data-driven state changes that might suspend (like router navigation or filtering massive arrays). Clean, synchronous layout changes (like opening a mobile menu or toggling a dropdown) should usually be handled via standard, urgent state updates

- The difference between Actions and regular event handling:
  https://react.dev/reference/react/useTransition#examples

- https://react.dev/reference/react/useTransition#exposing-action-props-from-components
  > TabButton component wraps its onClick logic in an action prop
  >
  > When you wrap the tab change logic inside `startTransition`, you tell React to switch its rendering engine from Synchronous Mode to Concurrent Mode.
  > 
  > Interruptible Rendering: React starts preparing the Posts tab UI in memory (the background). However, if the user suddenly clicks the About tab while React is halfway through rendering the slow posts, React says: "The user changed their mind. This background task is low-priority."

- https://react.dev/reference/react/useTransition#displaying-a-pending-visual-state
  
  `isPending` boolean value returned by useTransition to indicate to the user that a Transition is in progress.

- https://react.dev/reference/react/useTransition#preventing-unwanted-loading-indicators
  
  `PostsTab` component fetches some data using `use`. When you click the “Posts” tab, the `PostsTab` component suspends, causing `Suspense` to render. If you set state in `startTransition`, you can instead display the pending state
  > The Default Behavior (Without `useTransition`)
  > - You click the "Posts" tab.React instantly switches the current active tab state to 'posts'.
  > - React throws away the visible tab. React tries to render `<PostsTab />`. Because the data isn't ready yet, `<PostsTab />` suspends (it throws a Promise).
  > - React catches this Promise, climbs up the component tree to the closest `<Suspense>` boundary, and unmounts the entire active tab container to display the `fallback={<Loading />}` spinner.
  >
  > The Optimized Behavior (With `useTransition`)
  > - you tell React to switch its rendering engine into Concurrent Mode. While rendering the `<PostsTab />` in the background, React encounters the thrown data Promise. React freezes its background state, waiting patiently for the data Promise to resolve.

- https://react.dev/reference/react/useTransition#building-a-suspense-enabled-router
  > ```
  > ▲ Layout (Stays on screen)
  > └── 📦 Outer Suspense [fallback={<BigLoader />}]
  >      └── 📄 ArtistPage (The newly navigated page)
  >           ├── 👤 Biography (Fetches data via use())
  >           └── 📦 Inner Suspense [fallback={<AlbumsGlimmer />}]
  >                └── 🎵 Albums (Fetches data via use())
  > ```
  > The reason the outer `<Suspense fallback={<BigLoader />}>` is skipped while the inner `<Suspense fallback={<AlbumsGlimmer />}>` is shown comes down to one golden rule in React:
  > 
  > Transitions will only delay a fallback if that fallback is being newly revealed by the navigation itself.
  > - React hits the `Biography` component. The data for the biography isn't ready, so `Biography` suspends.
  > - React says: "Wait, if I show the BigLoader, I would have to hide the page the user is currently reading right now. That creates a bad UX."
  > - Once the `Biography` data resolves, React can successfully finish rendering the structural shell of the `ArtistPage`. React is now ready to commit the transition and reveal the `ArtistPage` to the user's screen.
  > - However, when React reaches the `Albums` component, `Albums` also suspends because its data isn't ready.
  > - React asks its Transition engine: "Does showing this `AlbumsGlimmer` force me to hide the old page the user was just looking at?"
  > - The answer is No! The `ArtistPage` shell (the heading, the biography text) is already built and ready to be mounted. The `AlbumsGlimmer` is safely nested inside the new page.

- https://react.dev/reference/react/useTransition#displaying-an-error-to-users-with-error-boundary
  
- https://react.dev/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition
  
  When you use `await` inside a `startTransition` function, the state updates that happen after each  `await` must wrap in a `startTransition` call:
  ```jsx
  startTransition(async () => {
    await someAsyncFunction();
    startTransition(() => {
      setPage('/about');
    });
  });
  ```

- https://react.dev/reference/react/useTransition#my-state-updates-in-transitions-are-out-of-order
  `updateQuantity` function simulates a request to the server. This function returns responses out of order. Actions within a Transition do not guarantee execution order.
  React provides higher-level abstractions like `useActionState` that handle ordering for you.
  ```jsx
    const [quantity, updateQuantityAction, isPending] = useActionState(
      async (prevState, payload) => {;
        const savedQuantity = await updateQuantity(payload);
        return savedQuantity; // Return the new quantity to update the state
      },
      1 // Initial quantity
    );

    return (
        <Item action={updateQuantityAction}/>
        <Total savedQuantity={quantity} isPending={isPending} />
    );
  ```

### 1.1.3. useActionState *

lets you update state with side effects using **Actions**.

```jsx
async function reducerAction(previousState, actionPayload) {
  const newState = await post(actionPayload);
  return newState;
}
const [state, dispatchAction, isPending] = useActionState(reducerAction, initialState);
```

React queues and executes multiple calls to dispatchAction sequentially. Each call to `reducerAction` receives the result of the previous call.

dispatchAction must be called from an Action (within transition).

If you set state after `await` in the `reducerAction` you currently need to wrap the state update in an additional `startTransition`.

- https://react.dev/reference/react/useActionState#adding-state-to-an-action
  
  Every time you click “Add Ticket,” React queues the action and shows the pending state until all the queued actions are resolved, and then re-renders with the final state.

> **Deep Dive**
> - Use `useReducer` to manage state of your UI. The reducer must be pure.
> - Use `useActionState` to manage state of your Actions. The reducer can perform side effects.
> 
> Both orders the call sequentially. If you want to perform Actions in parallel, use `useState` and `useTransition` directly.

- https://react.dev/reference/react/useActionState#using-with-useoptimistic
- https://react.dev/reference/react/useActionState#handling-errors

### 1.1.4. useOptimistic *

lets you optimistically update the UI. basically it lets you show a temporary value while an Action is in progress.

```jsx
const [optimisticState, setOptimistic] = useOptimistic(value, reducer?);
const [optimisticAge, setOptimisticAge] = useOptimistic(28);
```
- `value`: The value returned when there are no pending Actions.
- `optimisticState`: The current optimistic state. It is equal to `value` unless an Action is pending, in which case it is equal to the state returned by `reducer` (or the value passed to the `set function` if no `reducer` was provided).
- The `set function` that lets you update the optimistic state to a different value inside an Action.

The `set function` must be called inside an **Action**.

- https://react.dev/reference/react/useOptimistic#adding-optimistic-state-to-a-component
  ```jsx
  function MyComponent({age}) {
    const [optimisticAge, setOptimisticAge] = useOptimistic(age);
    function onAgeChange(e) {
      startTransition(async () => {
        setOptimisticAge(42);
        const newAge = await postAge(42);
        setAge(newAge);
      });
    }
  ```
  React will render the optimistic state `42` first while the `age` remains the current age. The Action waits for POST, and then renders the `newAge` for both `age` and `optimisticAge`. This state is called the “optimistic” because it is used to immediately present the user with the result of performing an Action, even though the Action actually takes time to complete.

- https://react.dev/reference/react/useOptimistic#using-optimistic-state-in-action-props
  ```jsx
  export default function EditName({ name, action }) {
    const [optimisticName, setOptimisticName] = useOptimistic(name);

    async function submitAction(formData) {
      const newName = formData.get('name');
      setOptimisticName(newName);

      const updatedName = await updateName(newName);
      startTransition(() => {
        action(updatedName);
      })
    }

    return (
      <form action={submitAction}>
        <p>Your name is: {optimisticName}</p>
        <p>
          <label>Change it: </label>
          <input type="text" name="name" disabled={name !== optimisticName} />
        </p>
      </form>
    );
  }
  ```

use reducer when 
- https://react.dev/reference/react/useOptimistic#updating-multiple-values-together
- https://react.dev/reference/react/useOptimistic#optimistically-adding-to-a-list
- https://react.dev/reference/react/useOptimistic#handling-multiple-action-types

https://react.dev/reference/react/useOptimistic#my-optimistic-updates-show-stale-values

https://react.dev/reference/react/useOptimistic#i-dont-know-if-my-optimistic-update-is-pending

### 1.1.5. [useDebugValue](https://react.dev/reference/react/useDebugValue)

### 1.1.6. useDeferredValue *

lets you defer updating a part of the UI i.e similar to transition tell React to treat an update as low-priority and render it in background.

```jsx
const deferredValue = useDeferredValue(value, initialValue?)
```
When `useDeferredValue` receives a different value, in addition to the current render (when it still uses the previous value), it schedules a re-render in the background with the new value.

When `initialValue` is provided, `useDeferredValue` will return it as value for the initial render of the component, and schedules a re-render in the background with the `deferredValue` returned.

- https://react.dev/reference/react/useDeferredValue#showing-stale-content-while-fresh-content-is-loading
  
  In this example, the `SearchResults` component suspends while fetching the search results. A loading fallback is shown while results are retrieved.

  A common alternative UI pattern is to defer updating the list of results and to keep showing the previous results until the new results are ready

  ```jsx
    const [query, setQuery] = useState('');
    const deferredQuery = useDeferredValue(query);

        <input value={query} onChange={e => setQuery(e.target.value)} />
  
        <Suspense fallback={<h2>Loading...</h2>}>
          <SearchResults query={deferredQuery} />
        </Suspense>
  ```
  when typing "a" then pause then "ab", instead of the Suspense fallback, you now see the stale result list until the new results have loaded
  - First, React re-renders with the new `query` (`"ab"`) but with the old `deferredQuery` (still `"a"`).
  - In the background, React tries to re-render with both `query` and `deferredQuery` updated to `"ab"`. If this re-render completes, React will show it on the screen. However, if it suspends (the results for `"ab"` have not loaded yet), React will abandon this rendering attempt, and retry this re-render again after the data has loaded.
  
  The deferred “background” rendering is interruptible. For example, if you type into the input again, React will abandon it and restart with the new value.

  Note that there is still a network request per each keystroke. What’s being deferred here is displaying results.

- https://react.dev/reference/react/useDeferredValue#indicating-that-the-content-is-stale
  ```jsx
  <div style={{ opacity: query !== deferredQuery ? 0.5 : 1, }}>
  ```
- https://react.dev/reference/react/useDeferredValue#deferring-re-rendering-for-a-part-of-the-ui
  
  Imagine you have a text field and a component `SlowList` that re-renders on every keystroke. the main performance problem is that whenever you type into the input, the `SlowList` receives new props, and re-rendering its entire tree makes the typing feel janky. `useDeferredValue` lets you prioritize updating the input (which must be fast) over updating the result list (which is allowed to be slower)

- https://react.dev/reference/react/useDeferredValue#how-is-deferring-a-value-different-from-debouncing-and-throttling

  If the work you’re optimizing doesn’t happen during rendering, debouncing and throttling are still useful. For example, they can let you fire fewer network requests. You can also use these techniques together.

### useEffect *

- https://react.dev/reference/react/useEffect#connecting-to-an-external-system
- https://react.dev/reference/react/useEffect#controlling-a-non-react-widget

### 1.1.7. useEffectEvent *

```jsx
const onEvent = useEffectEvent(callback)
```
Effect Events are a part of your Effect logic, but they behave more like an event handler. They always “see” the latest values from render (like props and state) without re-synchronizing your Effect, so they’re excluded from Effect dependencies. [See](./react.dev-learn.md#246-separating-events-from-effects)

- https://react.dev/reference/react/useEffectEvent#using-a-timer-with-latest-values
- https://react.dev/reference/react/useEffectEvent#using-an-event-listener-with-latest-values

### 1.1.8. [useId](https://react.dev/reference/react/useId)

### 1.1.9. [useImperativeHandle](https://react.dev/reference/react/useImperativeHandle)

### 1.1.10. [useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)

`useLayoutEffect` is a version of `useEffect` that fires before the browser repaints the screen.

### 1.1.11. useSyncExternalStore

lets you subscribe to an external store.
```jsx
const snapshot = useSyncExternalStore(subscribe, getSnapshot)
```
- `subscribe`: A function that takes a single callback argument and subscribes it to the store. When the store changes, it should invoke the provided callback, which will cause React to re-call `getSnapshot` and (if needed) re-render the component. The `subscribe` function should return a function that cleans up the subscription.
- `getSnapshot`: A function that returns a snapshot of the data in the store that’s needed by the component. 
  
The store `snapshot` returned by `getSnapshot` must be immutable.

If a different `subscribe` function is passed during a re-render, React will re-subscribe to the store using the newly passed `subscribe` function. You can prevent this by declaring `subscribe` outside the component.

for every Transition update, React will call `getSnapshot` a second time just before applying changes to the DOM. If it returns a different value than when it was called originally, React will restart the update from scratch, this time applying it as a blocking update.

It’s not recommended to suspend a render based on a store value returned by `useSyncExternalStore`. The reason is that mutations to the external store cannot be marked as non-blocking Transition updates, so they will trigger the nearest Suspense fallback.
```jsx
const LazyProductDetailPage = lazy(() => import('./ProductDetailPage.js'));

function ShoppingApp() {
  const selectedProductId = useSyncExternalStore(...);

  // ❌ Calling `use` with a Promise dependent on `selectedProductId`
  const data = use(fetchItem(selectedProductId))

  // ❌ Conditionally rendering a lazy component based on `selectedProductId`
  return selectedProductId != null ? <LazyProductDetailPage /> : <FeaturedProducts />;
}
```

- https://react.dev/reference/react/useSyncExternalStore#subscribing-to-an-external-store 
  ```jsx
  let nextId = 0;
  let todos = [{ id: nextId++, text: 'Todo #1' }];
  let listeners = [];

  export const todosStore = {
    addTodo() {
      todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
      emitChange();
    },
    subscribe(listener) {
      listeners = [...listeners, listener];
      return () => {
        listeners = listeners.filter(l => l !== listener);
      };
    },
    getSnapshot() {
      return todos;
    }
  };

  function emitChange() {
    for (let listener of listeners) {
      listener();
    }
  }

  export default function TodosApp() {
    const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);

    <button onClick={() => todosStore.addTodo()}>Add todo</button>
  ```
  When possible, use `useState` and `useReducer` instead. The `useSyncExternalStore` API is mostly useful if you need to integrate with existing non-React code.

- https://react.dev/reference/react/useSyncExternalStore#subscribing-to-a-browser-api
  
  [See](./react.dev-learn.md#244-you-might-not-need-an-effect)

## 1.2. Components

### 1.2.1. `<Suspense>`

display a fallback until its children have finished loading.

Only Suspense-enabled data sources will activate the Suspense component. They include:
- Data fetching with Suspense-enabled frameworks like `Relay` and `Next.js`
- Lazy-loading component code with `lazy`
- Reading the value of a cached Promise with `use`

By default, the whole tree inside Suspense is treated as a single unit. For example, even if only one of these components suspends waiting for some data, all of them together will be replaced by the loading indicator:
```jsx
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```

- https://react.dev/reference/react/Suspense#resetting-suspense-boundaries-on-navigation


### 1.2.2. `<Activity>`

lets you hide/restore UI and internal state of its children.

```jsx
<Activity mode={isShowingSidebar ? "visible" : "hidden"}>
  <Sidebar />
</Activity>
```
When an Activity boundary is `hidden`, React will visually hide its children using the `display: "none"` CSS property. It will also destroy their Effects.

While hidden, children still re-render in response to new props, albeit at a lower priority than the rest of the content.

When the boundary becomes visible again, React will reveal the children with their previous state restored, and re-create their Effects.

- https://react.dev/reference/react/Activity#restoring-the-dom-of-hidden-components
- https://react.dev/reference/react/Activity#pre-rendering-content-thats-likely-to-become-visible
- https://react.dev/reference/react/Activity#my-hidden-components-have-unwanted-side-effects

## 1.3. APIs

### 1.3.1. use
lets you read the value of a resource like a cached Promise or context
```jsx
const value = use(resource);
```
Unlike React Hooks, `use` can be called within loops and conditional statements like if. Like React Hooks, the function that calls use must be a Component or Hook. 

When called with a Promise, the use API integrates with `Suspense` and `Error Boundaries`. If the component that calls use is wrapped in a `Suspense` boundary, the fallback will be displayed. If the Promise passed to `use` is rejected, the fallback of the nearest Error Boundary will be displayed.

- https://react.dev/reference/react/use#reading-context-with-use
  
  When a context is passed to use, it works similarly to `useContext`. While `useContext` must be called at the top level of your component, `use` can be called inside conditionals like `if` and loops like `for`.

# 2. react-dom@19.2

## 2.1. Hooks

### 2.1.1. [useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus)

gives you status information for a parent `<form>`
```jsx
const { pending, data, method, action } = useFormStatus();
```

- https://react.dev/reference/react-dom/hooks/useFormStatus#display-a-pending-state-during-form-submission
  ```jsx
  function Submit() {
    const { pending } = useFormStatus();
    return <button disabled={pending}>...</button>;
  }

    <form action={submit}>
      <Submit />
    </form>
  ```

- https://react.dev/reference/react-dom/hooks/useFormStatus#read-form-data-being-submitted

## 2.2. Components

### 2.2.1. `<form>`

```jsx
<form action={search}>
</form>
```
- `action`: a URL or function. When a function is passed to action the function will handle the form submission in a Transition following the Action prop pattern. The function passed to action may be async and will be called with a single argument containing the form data of the submitted form. The action prop can be overridden by a formAction attribute on a `<button>`, `<input type="submit">`, or `<input type="image">` component.

- https://react.dev/reference/react-dom/components/form#handle-form-submission-on-the-client
  
  After the `action` function succeeds, all uncontrolled field elements in the form are reset.
  ```jsx
  export default function Search() {
    function search(formData) {
      const query = formData.get("query");
      alert(`You searched for '${query}'`);
    }
    return (
      <form action={search}>
        <input name="query" />
        <button type="submit">Search</button>
      </form>
    );
  }
  ```

- https://react.dev/reference/react-dom/components/form#display-a-pending-state-during-form-submission
  
  with `useFormStatus`
  
- https://react.dev/reference/react-dom/components/form#optimistically-updating-form-data
  
  with `useOptimistic`

- https://react.dev/reference/react-dom/components/form#handling-form-submission-errors
- https://react.dev/reference/react-dom/components/form#handling-multiple-submission-types
  
  Each button inside a form can be associated with a distinct action or behavior by setting the `formAction` prop

### 2.2.2. `<input>`

- https://react.dev/reference/react-dom/components/input#optimizing-re-rendering-on-every-keystroke
  ```jsx
  // before
  <form>
    <input value={firstName} onChange={e => setFirstName(e.target.value)} />
  </form>
  <PageContent />
  // after
  <SignupForm /> // only SignupForm re-renders on every keystroke
  <PageContent />
  ```

## 2.3. APIs

### 2.3.1. createPortal, flushSync, 
### 2.3.2. preconnect, prefetchDNS,  
### 2.3.3. preinit, preinitModule
### 2.3.4. preloadModule, preload

# 3. Rules of React

- Components and Hooks must be pure 
  - React components are assumed to always return the same output with respect to their inputs – props, state, and context.
  - Side effects must run outside of render
  - Props and state are immutable
  - Return values and arguments to Hooks are immutable
  - Values are immutable after being passed to JSX
- React calls Components and Hooks