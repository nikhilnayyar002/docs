v19.2

- [1. GET STARTED](#1-get-started)
  - [1.1. Quick Start](#11-quick-start)
    - [1.1.1. Quick Start](#111-quick-start)
    - [1.1.2. Tutorial: Tic-Tac-Toe](#112-tutorial-tic-tac-toe)
  - [1.2. Installation](#12-installation)
    - [1.2.1. Creating a React App](#121-creating-a-react-app)
    - [1.2.2. Build a React app from Scratch](#122-build-a-react-app-from-scratch)
    - [1.2.3. Add React to an Existing Project](#123-add-react-to-an-existing-project)
  - [1.3. Setup](#13-setup)
    - [1.3.1. Editor Setup](#131-editor-setup)
    - [1.3.2. Using TypeScript](#132-using-typescript)
    - [1.3.3. React Developer Tools](#133-react-developer-tools)
  - [1.4. React Compiler](#14-react-compiler)
    - [1.4.1. React Compiler](#141-react-compiler)
    - [1.4.2. Introduction](#142-introduction)
    - [1.4.3. Installation](#143-installation)
    - [1.4.4. Incremental Adoption](#144-incremental-adoption)
    - [1.4.5. Debugging and Troubleshooting](#145-debugging-and-troubleshooting)
- [2. LEARN REACT](#2-learn-react)
  - [2.1. Describing the UI](#21-describing-the-ui)
    - [2.1.1. Passing Props to a Component](#211-passing-props-to-a-component)
    - [2.1.2. Conditional Rendering](#212-conditional-rendering)
    - [2.1.3. Keeping Components Pure](#213-keeping-components-pure)
  - [2.2. Adding Interactivity](#22-adding-interactivity)
    - [2.2.1. Responding to Events](#221-responding-to-events)
    - [2.2.2. State: A Component's Memory](#222-state-a-components-memory)
    - [2.2.3. Render and Commit](#223-render-and-commit)
    - [2.2.4. State as a Snapshot](#224-state-as-a-snapshot)
    - [2.2.5. Queueing a Series of State Updates](#225-queueing-a-series-of-state-updates)
    - [2.2.6. Updating Objects in State](#226-updating-objects-in-state)
    - [2.2.7. Updating Arrays in State](#227-updating-arrays-in-state)
  - [2.3. Managing State](#23-managing-state)
    - [2.3.1. Reacting to Input with State](#231-reacting-to-input-with-state)
    - [2.3.2. Sharing State Between Components](#232-sharing-state-between-components)
    - [2.3.3. Extracting State Logic into a Reducer](#233-extracting-state-logic-into-a-reducer)
    - [2.3.4. Passing Data Deeply with Context](#234-passing-data-deeply-with-context)
    - [2.3.5. Scaling Up with Reducer and Context](#235-scaling-up-with-reducer-and-context)
  - [2.4. Escape Hatches](#24-escape-hatches)
    - [2.4.1. Referencing Values with Refs](#241-referencing-values-with-refs)
    - [2.4.2. Manipulating the DOM with Refs](#242-manipulating-the-dom-with-refs)
    - [2.4.3. Synchronizing with Effects](#243-synchronizing-with-effects)
    - [2.4.4. You Might Not Need an Effect](#244-you-might-not-need-an-effect)
    - [2.4.5. Lifecycle of Reactive Effects](#245-lifecycle-of-reactive-effects)
    - [2.4.6. Separating Events from Effects](#246-separating-events-from-effects)


# 1. GET STARTED

## 1.1. Quick Start

### 1.1.1. Quick Start

```jsx
// component
function MyButton({ count, onClick } // props) { 
  // returning JSX
  return (<button onClick={onClick}>{count}</button>);
}
export default function MyApp() {
  const [count, setCount] = useState(0); // Hooks 

  const listItems = products.map(product =>
    <li key={product.id}>
      {product.title}
    </li>
  );

  return (
    isLoggedIn && (
        <>
          <h1>{user.name}</h1>
          <img className="avatar" src={user.imageUrl} style={{ width: user.imageSize }}/>
          <ul>{listItems}</ul>
          <MyButton count={count} onClick={()=>setCount(count + 1) />
        </>
    )
  );
}
```

### 1.1.2. [Tutorial: Tic-Tac-Toe](https://react.dev/learn/tutorial-tic-tac-toe)
...

## 1.2. Installation

### 1.2.1. [Creating a React App](https://react.dev/learn/creating-a-react-app)
...

### 1.2.2. [Build a React app from Scratch](https://react.dev/learn/build-a-react-app-from-scratch)
...

### 1.2.3. [Add React to an Existing Project](https://react.dev/learn/add-react-to-an-existing-project)
...

## 1.3. Setup

### 1.3.1. [Editor Setup](https://react.dev/learn/editor-setup)
...

### 1.3.2. [Using TypeScript](https://react.dev/learn/typescript)
...

### 1.3.3. [React Developer Tools](https://react.dev/learn/react-developer-tools)
...

## 1.4. React Compiler

### 1.4.1. [React Compiler](https://react.dev/learn/react-compiler)
...

### 1.4.2. Introduction

> My Note - [youtube](https://www.youtube.com/watch?v=40osg7LoShc)

...

### 1.4.3. Installation
...

### 1.4.4. Incremental Adoption
...

### 1.4.5. Debugging and Troubleshooting

**Compiler Errors vs Runtime Issues**

*Compiler errors* occur at build time are rare because the compiler is designed to skip problematic code rather than fail.

Most of the time, if you encounter an issue with React Compiler, it’s a *runtime issue*. This typically happens when your code violates the Rules of React in subtle ways that the compiler couldn’t detect, and the compiler mistakenly compiled a component it should have skipped.

When debugging runtime issues, focus your efforts on finding Rules of React violations in the affected components that were not detected by the ESLint rule.

** Temporarily Disable Compilation**

```jsx
function ProblematicComponent() {
  "use no memo"; // Skip compilation for this component
  // ... rest of component
}
```

# 2. LEARN REACT

## 2.1. Describing the UI

### 2.1.1. Passing Props to a Component

```jsx
// Specifying a default value for a prop 
function Avatar({ person, size = 100 }) {} 

// Forwarding props with the JSX spread syntax 
<Avatar {...props} />

// Passing JSX as children
<Card>
  <Avatar />
</Card>
function Card({ children }) {
  return (<div className="card">{children}</div>);
}
```

### 2.1.2. Conditional Rendering

**Conditionally returning nothing with null**

In practice, returning `null` from a component isn’t common because it might surprise a developer trying to render it. More often, you would conditionally include or exclude the component in the parent component’s JSX.

**Logical AND operator (`&&`)**

A JavaScript `&&` expression returns the value of its right if the left is `true`. React considers `false` as a “hole” in the JSX tree, just like `null` or `undefined`, and doesn’t render anything in its place. Don’t put numbers on the left side (React will happily render 0).

### 2.1.3. Keeping Components Pure

- *It minds its own business*. It does not change any objects or variables that existed before it was called.
- *Same inputs, same output*. Given the same inputs, a pure function should always return the same result.

> Deep Dive
> **Detecting impure calculations with StrictMode**
>
> in React there are three kinds of inputs that you can read while rendering: props, state, and context. You should always treat these inputs as read-only.
> When you want to change something in response to user input, you should set state instead of writing to a variable. 
>
> React offers a “Strict Mode” in which it calls each component’s function twice during development.

**Local mutation: Your component’s little secret**

it’s completely fine to change variables and objects that you’ve just created while rendering inside the component (and doesnt effect outside the component). 

```jsx
export default function TeaGathering() {
  const cups = [];
  for (let i = 1; i <= 12; i++) cups.push(<Cup key={i} guest={i} />);
  return cups;
}
```

## 2.2. Adding Interactivity

### 2.2.1. Responding to Events

```jsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (<button onClick={handleClick}> Click me </button>);
}
```

Event handler functions:
- Are usually defined inside your components.
- Have names that start with handle, followed by the name of the event.

you can define an event handler inline in the JSX:

```jsx
<button onClick={() => alert('You clicked me!')}>
```

**Naming event handler props**

By convention, event handler props should start with `on`, followed by a capital letter.

> Deep Dive 
> **Capture phase events**
>
> In rare cases, you might need to catch all events on child elements, even if they stopped propagation. You can do this by adding `Capture` at the end of the event name:
> ```jsx
> <div onClickCapture={() => { /* this runs first */ }}>
>  <button onClick={e => e.stopPropagation()} />
>  <button onClick={e => e.stopPropagation()} />
> </div>

### 2.2.2. State: A Component's Memory

> Hooks—functions starting with `use`—can only be called at the top level of your components or your own Hooks. You can’t call Hooks inside conditions, loops, or other nested functions.

### 2.2.3. Render and Commit

process of requesting and serving UI has three steps:

**Step 1: Trigger a render**
- It’s the component’s initial render.
- The component’s (or one of its ancestors’) state has been updated.

```jsx
const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

**Step 2: React renders your components**

For subsequent renders, React will call the function component whose state update triggered the render. This process is recursive: if the updated component returns some other component, React will render that component and so on.

**Step 3: React commits changes to the DOM**

*React only changes the DOM nodes if there’s a difference between renders.* 

### 2.2.4. State as a Snapshot

*Setting state only changes it for the next render.* 

```jsx
  const [number, setNumber] = useState(0);
  return (
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
  )
```

Even though you called `setNumber(number + 1)` three times, in this render’s event handler number is always `0`, so you set the state to `1` three times. 

**State over time**

```jsx
  const [number, setNumber] = useState(0);
  return (
      <button onClick={() => {
        setNumber(number + 5);
        alert(number);
      }}>+5</button>
  )
```

A state variable’s value never changes within a render, even if its event handler’s code is asynchronous. Inside that render’s `onClick`, the value of number continues to be `0` even after `setNumber(number + 5)` was called. Its value was “fixed” when React “took the snapshot” of the UI by calling your component. React keeps the state values “fixed” within one render’s event handlers.

### 2.2.5. Queueing a Series of State Updates

It is an uncommon use case, but if you would like to update the same state variable multiple times before the next render, instead of passing the next state value like `setNumber(number + 1)`, you can pass a function that calculates the next state based on the previous one in the queue, like `setNumber(n => n + 1)`.

### 2.2.6. Updating Objects in State

```js
setPerson({
  ...person,
  artwork: { ...person.artwork, city: 'New Delhi' }
});
```

### 2.2.7. [Updating Arrays in State](https://react.dev/learn/updating-arrays-in-state)

...

## 2.3. Managing State

### 2.3.1. Reacting to Input with State

```jsx
  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }
  ...
  <form onSubmit={handleSubmit}>
```

### 2.3.2. Sharing State Between Components

```jsx
  {showB && <Counter />} 
```

*React preserves a component’s state for as long as it’s being rendered at its position in the UI tree.* If it gets removed, or a different component gets rendered at the same position, React discards its state.

**Same component at the same position preserves state**

```jsx
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
```

Whether `isFancy` is `true` or `false`, you always have a `<Counter />` as the first child of the `div` returned from the root App component. 

**Resetting state at the same position**

Option 1: Rendering a component in different positions

```jsx
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
```
...

Option 2: Resetting state with a key 

```jsx
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
```
...

**Resetting a form with a key**

In this chat app, the `<Chat>` component contains the text input state. Try entering something into the input, and then press “Alice” or “Bob” to choose a different recipient. You will notice that the input state is preserved because the `<Chat>` is rendered at the same position in the tree. To fix it

```jsx
<Chat key={to.id} contact={to} />
```

### 2.3.3. Extracting State Logic into a Reducer

```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // "action" object:
    {
      type: 'deleted',
      id: taskId,
    }
  );
}

function tasksReducer(tasks, action) {
  if (action.type === 'added') 
   ...
  else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}

const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

### 2.3.4. Passing Data Deeply with Context

```jsx
export const LevelContext = createContext(1);

export default function Heading({  }) {
  const level = useContext(LevelContext);
  // ...
}

export default function Section({ level, children }) {
  return (
      <LevelContext value={level}>
        {children}
      </LevelContext>
  );
}

    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
      </Section>
    </Section>
```
`Heading` asks the closest value of `LevelContext` above with `useContext(LevelContext)`

**Using and providing context from the same component**

Currently, you still have to specify each section’s `level` manually. Since context lets you read information from a component above, each `Section` could read the `level` from the `Section` above

```jsx
export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
```

**Before you use context**

Extract components and pass JSX as children to them. If you pass some data through many layers of intermediate components that don’t use that data this often means that you forgot to extract some components along the way. For example, maybe you pass data props like `posts` to visual components that don’t use them directly, like `<Layout posts={posts} />`. Instead, make Layout take children as a prop, and render `<Layout><Posts posts={posts} /></Layout>`.

**Use cases for context**

...

Context is not limited to static values. If you pass a different value on the next render, React will update all the components reading it below! This is why context is often used in combination with state.

### 2.3.5. Scaling Up with Reducer and Context

...

Currently, the tasks state and the dispatch function are only available in the top-level TaskApp component. To let other components read the list of tasks or change it, you have to explicitly pass down the current state and the event handlers that change it as props.

```jsx
export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        ...
      </TasksDispatchContext>
    </TasksContext>
  );
}
```

**Moving all wiring into a single file**

```jsx
// TasksContext.js
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

// App.js
export default function TaskApp() {
  return (
    <TasksProvider>
      ...
    </TasksProvider>
  );
}

// in other components
const tasks = useTasks();
const dispatch = useTasksDispatch();
```
Now all of the context and reducer wiring is in TasksContext.js. This keeps the components clean and uncluttered, focused on what they display rather than where they get the data

## 2.4. Escape Hatches

### 2.4.1. Referencing Values with Refs

When you want a component to “remember” some information, but you don’t want that information to trigger new renders, you can use a ref.

```jsx
const ref = useRef(0);
```

### 2.4.2. Manipulating the DOM with Refs

```jsx
 const inputRef = useRef(null);
     <input ref={inputRef} />
inputRef.current.focus(); 
```

> Deep Dive
> **How to manage a list of refs using a ref callback**
> 
> pass a function to the `ref` attribute. This is called a ref callback. React will call your ref callback with the DOM node when it’s time to set the ref, and call the cleanup function returned from the callback when it’s time to clear it.
>
> ```jsx
> export default function CatFriends() {
>   const itemsRef = useRef(null);
>   const [catList, setCatList] = useState(...);
> 
>   function scrollToCat(cat) {
>     const map = getMap(); const node = map.get(cat); node.scrollIntoView({ ... });
>   }
> 
>   function getMap() {
>     if (!itemsRef.current)
>       itemsRef.current = new Map(); // Initialize the Map on first usage.
>     return itemsRef.current;
>   }
> 
>   return (
>      ...
>         <button onClick={() => scrollToCat(catList[0])}>Neo</button>
>      ...
>         <ul>
>           {catList.map((cat) => (
>             <li
>               key={cat.id}
>               ref={(node) => {
>                 const map = getMap();
>                 map.set(cat, node);
> 
>                 return () => {
>                   map.delete(cat);
>                 };
>               }}
>             ></li>
>           ))}
>         </ul>
>   );
> }
> ```

**Accessing another component’s DOM nodes**

```jsx
function MyInput({ ref }) {
  return <input ref={ref} />;
}

function MyForm() {
  const inputRef = useRef(null);
  return <MyInput ref={inputRef} />
}
```

> Deep Dive
> **Exposing a subset of the API with an imperative handle**
> ```jsx
> function MyInput({ ref }) {
>   const realInputRef = useRef(null);
>   useImperativeHandle(ref, () => ({
>     // Only expose focus and nothing else
>     focus() {
>       realInputRef.current.focus();
>     },
>   }));
>   return <input ref={realInputRef} />;
> };
> ```

**When React attaches the refs**

During the first render, the DOM nodes have not yet been created, so ref.current will be null. And during the rendering of updates, the DOM nodes haven’t been updated yet. So it’s too early to read them.

React sets `ref.current` during the commit. Before updating the DOM, React sets the affected `ref.current` values to null. After updating the DOM, React immediately sets them to the corresponding DOM nodes.

> Deep Dive
> **[Flushing state updates synchronously with flushSync](https://react.dev/learn/manipulating-the-dom-with-refs#flushing-state-updates-synchronously-with-flush-sync)**
>
> ...
> ```jsx
> flushSync(() => {
>   setTodos([ ...todos, newTodo]);
> });
> listRef.current.lastChild.scrollIntoView();
> ```
> This will instruct React to update the DOM synchronously right after the code wrapped in `flushSync` executes. the last todo will already be in the DOM by the time you try to scroll to it.

### 2.4.3. Synchronizing with Effects

Effects run at the end of a commit after the screen updates. 

**Step 1: Declare an Effect**

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  if (isPlaying) ref.current.play();
  else ref.current.pause();

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
  );
}
```
The reason this code isn’t correct is that it tries to do something with the DOM node during rendering. In React, rendering should be a pure calculation of JSX and should not contain side effects like modifying the DOM.

The solution here is to wrap the side effect with useEffect to move it out of the rendering calculation:
```jsx
  useEffect(() => {
    if (isPlaying) ref.current.play();
    else ref.current.pause();
  });
}
```

in development React remounts every component once immediately after its initial mount. Remounting components only happens in development to help you find Effects that need cleanup. You can turn off Strict Mode to opt out.

**Fetching data**

If your Effect fetches something, the cleanup function should either abort the fetch or ignore its result:

```jsx
useEffect(() => {
  let ignore = false;
  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) setTodos(json);
  }
  startFetching();

  return () => ignore = true;
}, [userId]);
```
consider extracting your fetching logic into a custom Hook.

> Deep Dive
> **What are good alternatives to data fetching in Effects?**
>
> Writing fetch calls inside Effects has significant downsides:
> - Effects don’t run on the server.
> - Fetching directly in Effects makes it easy to create “network waterfalls”. You render the parent component, it fetches some data, renders the child components, and then they start fetching their data. If the network is not very fast, this is significantly slower than fetching all data in parallel.
> - Fetching directly in Effects usually means you don’t preload or cache data
>
> we recommend the following approaches:
> - If you use a framework, use its built-in data fetching mechanism
> -  consider using or building a client-side cache. Popular open source solutions include [TanStack Query](https://tanstack.com/query/latesthttps://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/), and [React Router 6.4+](https://beta.reactrouter.com/en/main/start/overview).
>

**Not an Effect: Initializing the application**

Some logic should only run once when the application starts. You can put it outside your components:

```js
if (typeof window !== 'undefined') { // Check if we're running in the browser.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {}
```

### 2.4.4. You Might Not Need an Effect

**Resetting all state when a prop changes**

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
}
```
This is inefficient because `ProfilePage` and its children will first render with the stale value, and then render again.

Instead, you can tell React that each user’s profile is conceptually a different profile by giving it an explicit key. Split your component:

```jsx
export default function ProfilePage({ userId }) {
  return (<Profile userId={userId} key={userId}/>);
}
function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
}
```
Normally, React preserves the state when the same component is rendered in the same spot. By passing userId as a key to the Profile component, you’re asking React to treat two Profile components with different userId as two different components that should not share any state. 

**Adjusting some state when a prop changes**

```jsx
  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
```
Every time the `items` change, the `List` and its child components will render with a stale selection value at first. Then React will update the DOM and run the Effects. Finally, the `setSelection(null)` call will cause another re-render of the `List` and its child components, restarting this whole process again.

```jsx
  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
```
In the above example, `setSelection` is called directly during a render.

When you update a component during rendering, React throws away the returned JSX and immediately retries rendering. 

Although this pattern is more efficient than an Effect, most components shouldn’t need it either. adjusting state based on props or other state makes your data flow more difficult to understand and debug. instead of storing (and resetting) the selected item, you can store the selected item ID:
```jsx
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
```

**Subscribing to an external store**

Sometimes, your components may need to subscribe to some data outside of the React state. This data could be from a third-party library or a built-in browser API. Since this data can change without React’s knowledge, you need to manually subscribe your components to it. This is often done with an Effect:

```jsx
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
```
React has a purpose-built Hook for subscribing to an external store that is preferred instead
```jsx
function subscribe(callback) { // call this callback to let react know store has changed
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => { // cleanup
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // How to get the value on the client
  );
}
...
 const isOnline = useOnlineStatus();
```

### 2.4.5. Lifecycle of Reactive Effects

**All variables declared in the component body are reactive**

Props and state aren’t the only reactive values. Values that you calculate from them are also reactive. 

> Deep Dive
> **Can global or mutable values be dependencies?**
>
> A mutable value like location.pathname can’t be a dependency. It’s mutable, so it can change at any time completely outside of the React rendering data flow. Changing it wouldn’t trigger a re-render of your component.
>
> A mutable value like `ref.current` or things you read from it also can’t be a dependency. The ref object returned by `useRef` itself can be a dependency, but its `current` property is intentionally mutable.

**What to do when you don’t want to re-synchronize**

you could “prove” to the linter that these values aren’t reactive values, i.e. that they can’t change as a result of a re-render. don’t depend on rendering and always have the same values, you can move them outside the component. You can also move them inside the Effect. They aren’t calculated during rendering, so they’re not reactive

### 2.4.6. Separating Events from Effects

Event handlers run in response to specific interactions

Effects run whenever synchronization is needed 

**Reactive values and reactive logic**

Logic inside event handlers is not reactive. It will not run again unless the user performs the same interaction (e.g. a click) again. Event handlers can read reactive values without “reacting” to their changes.

Logic inside Effects is reactive. If your Effect reads a reactive value, you have to specify it as a dependency. Then, if a re-render causes that value to change, React will re-run your Effect’s logic with the new value.

**Extracting non-reactive logic out of Effects**

imagine that you want to show a notification when the user connects to the chat. You read the current theme (dark or light) from the props so that you can show the notification in the correct color:

```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme); // you don’t want this line to be reactive
    });
```

However, `theme` is a reactive value, and every reactive value read by an Effect must be declared as its dependency. chat re-connects every time you switch between the dark and the light theme. That’s not great!

Use a special Hook called `useEffectEvent` to extract this non-reactive logic out of your Effect:

```jsx
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  useEffect(() => { ...
    connection.on('connected', () => {
      onConnected();
    }); ...
  }, [roomId]); // ✅ All dependencies declared
```