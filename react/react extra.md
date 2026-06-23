
- https://react.dev/blog/2024/12/05/react-19
  
  - https://react.dev/blog/2024/12/05/react-19#support-for-metadata-tags
    
    React automatically hoist `<title> <link> and <meta>` tags, to the <head> section of document.

  - https://react.dev/blog/2024/12/05/react-19#support-for-stylesheets
    
     If you tell React the `precedence` of your stylesheet it will manage the insertion order of the stylesheet in the DOM and during rendering will wait for newly rendered stylesheets to load before committing the render.
     ```jsx
        <Suspense fallback="loading...">
          <link rel="stylesheet" href="foo" precedence="default" />
          <link rel="stylesheet" href="bar" precedence="high" />
          <article class="foo-class bar-class">...</article>
        </Suspense>
     ```
   - https://react.dev/blog/2024/12/05/react-19#support-for-async-scripts

      better support for async scripts by allowing you to render them anywhere in your component tree, inside the components that actually depend on the script

   - https://react.dev/blog/2024/12/05/react-19#support-for-preloading-resources

     These APIs can be used to optimize initial page loads by moving discovery of additional resources like fonts out of stylesheet loading. They can also make client updates faster by prefetching a list of resources used by an anticipated navigation and then eagerly preloading those resources on click or even on hover.

- https://react.dev/reference/react/useCallback#should-you-add-usecallback-everywhere
  
  you can make a lot of memoization unnecessary by following a few principles:
  - When a component visually wraps other components, let it accept JSX as children. Then, if the wrapper component updates its own state, React knows that its children don’t need to re-render.
    ```jsx
    // ❌ The Sub-Optimal Way
    function VisualWrapper() {
      const [count, setCount] = useState(0);
      return (
        <div className="wrapper">
          <button onClick={() => setCount(count + 1)}>Count: {count}</button>
          <HeavyComponent /> 
        </div>
      );
    }

    // ✅ The Highly Optimized Way
    function VisualWrapper({ children }) {
      ...
      return (
        <div className="wrapper">
          ...
          {children} 
        </div>
      );
    }
    <VisualWrapper>
      <HeavyComponent />
    </VisualWrapper>
    ```
  - Prefer local state and don’t lift state up any further than necessary. Don’t keep transient state like forms and whether an item is hovered at the top of your tree or in a global state library.

     "Transient state" refers to temporary, fast-changing UI states—such as tracking which dropdown is open, which button is being hovered, or the characters a user is actively typing into a form input box.
     If you store this transient state in a global store (like Redux) or lift it to the root component of your layout tree, you create a massive performance bottleneck.

   - Avoid unnecessary Effects that update state. Most performance problems in React apps are caused by chains of updates originating from Effects that cause your components to render over and over.
   - [rules of react](./react.dev-reference.md#3-rules-of-react)


