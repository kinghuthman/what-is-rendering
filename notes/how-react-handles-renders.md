# How does react handle renders

## Queue Renders

After the initial render is completed, react has a few different ways to queue re-renders

- Functional Components
  - useState setters
  - useReducer dispatches
- Class Components
  - this.setState()
  - this.forceUpdate()
- Other
  - calling the ReactDom top level render(<App>) method again which is similar to forceUpdate() on the root component

Note that function components don't have a force update method, but you can get the same behavior by using a useReducer hook that always increments a counter: `const [, forceRender] = useReducer((c) c+ 1, 0)`

## Standard Render Behavior

React's default behavior, when a parent component renders, its child components will recursively render too

As an example: A > B > C > D

- The user clicks a button that increments a counter in component B
  - we call setState in component B which requeues a rerender of B
    - React starts the render past from the top of the tree
      - React sees that A is not in need of an update so passes on
        - React sees that B is in need of an update and renders it. B returns <C /> as it did last time
          - C was not marked as needing an update but because it's a parent B did, it will now re-render as React moves downward. C returns <D />
            - D was also not marked for rendering but because C did, react moves downward and re-renders D too

To keep it simple, re-rendering a component re-renders all components within it as well. It doesn't care if props change, if the parent re-renders, it will re-render the child as well.

This means that calling setState() in your root <App> component will cause every component in the component tree to re-render.

Rendering is not a bad thing - it's how react knows whether it actually needs to make changes to the DOM

## Rules of React Rendering

One of the rules of react rendering is that it must be "pure" and not have any side effects!

- This can be tricky and confusing, because many side effects don't break anything.
  - Strictly speaking a console.log() is a side effect
  - Mutating a prop is a side effect and it might not break anything
  - Making an AJAX call in the middle of rerendering is definitely a side effect and can definitely cause unexpected app behavior

### Sebastian Markbage "Rules of React"

He defines the expected behavior for react lifecycle methods, including render. https://gist.github.com/sebmarkbage/75f0838967cd003cd7f9ab938eb1958f

- Render logic must not:

  - can't mutate existing variables and objects
  - can't create random values like Math.random() or Date.now()
  - can't make network requests
  - can't queue state updates

- Render logic may:
  - mutate objects that were newly created while rendering
  - throw errors
  - "lazy initialize" data that hasn't been created yet, such as a cached value

## Component Metadata and Fiber

React stores an internal data structure that tracks all the current component instances that exist in the application. The core piece of this data structure is an object called a "fiber", which contains metadata fields that describe:

- What component type is supposed to be rendered at this point in the component tree
- the current props and state associated with this component
- Pointers to parent, sibling and child components
- Other internal metadata that React uses to track the rendering process

Released in React version 16.0
![React Fiber type](https://github.com/user-attachments/assets/8b7e886e-32dd-47bd-ad3c-9de8442f201c)

- Full definition of Fiber type as of React 18
  - https://github.com/facebook/react/blob/v18.0.0/packages/react-reconciler/src/ReactInternalTypes.js#L64-L193

During a rendering pass, React will iterate over this tree of fiber objects and construct an updated tree as it calculates the new rendering results

- Note that these "fiber" objects store the real component props and state values. When you use `props` and `state` in your components, React is actually access to the values that were stored on the fiber objects. In fact, for class components, [react copies componentInstance.props = newProps over to the component right before rendering it](https://github.com/facebook/react/blob/v18.0.0/packages/react-reconciler/src/ReactFiberClassComponent.new.js#L1083-L1087). So, this.props does exist, but it only exists because react copied the reference over from its internal data structures. In that sense, components are sort of a facade over React's fiber objects.

Similarly, React hooks work [because React stores all of the hooks for a component as a linked list attached to that component's fiber object](https://www.swyx.io/hooks). When react renders a function component, it gets that linked list of hook description entries from the fiber, and every time you call another hook, [it returns the appropriate values that were stored in the hook description object (like the `state` and `dispatch` values for `useReducer`)](https://github.com/facebook/react/blob/v18.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L908).

When a parent component renders a given child component for the first time, React creates a fiber object to track that instance of component. For class components, it literally calls `const instance = new YourComponentType(props) and saves the actual component instance onto the fiber object. For function components, React just calls YourComponentType(props) as a function.

## Component Types and Reconciliation

As described in the ["Reconciliation" docs page](http://legacy.reactjs.org/docs/reconciliation.html#elements-of-different-types), React tries to be efficient during re-renders, by reusing as much of the existing component tree and DOM structure as possible. If you ask React to render the same type of component or HTML node in the same place in the tree, React will reuse that and just appluy updates if appropriate, instead of re-creating it from scratch. That means that React will keep component instances alive as long as you keep asking React to render that component type in the same place. For class components, it actually does use the same actual instance of your component. A function component has no true "instance" the way a class does, but we can think of <MyFunctionComponent /> as representing an "instance" in terms of a "component of this type is being shown here and kept alive".

So, how does React know when and how the output has actually changed?

React's rendering logic compares elements based on their `type` field first, using `===` reference comparisons. If an element in a given spot has changed to a different type, such as going from a <div> to <span> or <ComponentA> to <ComponentB>, react will speed up the comparison process by assuming that entire tree has changed. As a result, React will destroy that entire existing component tree section, including al DOM nodes, and recreate it from scratch with new component instances.

This means that you must never create new component types while rendering! Whenever you create a new component type, it's a different reference, and that will cause React to repeatedly destroy and recreate the child component tree.

![Don't recreate component](https://github.com/user-attachments/assets/a34af1db-dbd2-479e-8df6-354fdf07f47a)

- A new `ChildComponent` function is created every time `ParentComponent` renders
- This breaks referential equality, which can cause:
  - Unnecessary re-renders of memoized components
  - Invalid behavior with hooks like useMemo, useCallback, or React.memo
  - Stale closures or incorrect behavior if ChilcComponent uses props or context

Best Practice:

- Define `ChildComponent` outside of `ParentComponent` to ensure it's referentially stable

### When is it acceptable?

- ChildComponent is extremely simple and doesn't rely on props/state
- You're not memoizing anything or caring about render performance
- It's used only once and never conditionally
- You're intentionally using a closure for specific scoping logic (unusual in UI)

Even then, for clarity, reusability, and maintainability, it's better to define components outside.

## Keys and Reconciliation
