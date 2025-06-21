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
