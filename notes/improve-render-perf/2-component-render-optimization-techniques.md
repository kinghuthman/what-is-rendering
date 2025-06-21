## Component Render optimization Techniques

React offers three primary APIs that allow us to potentially skip rendering a component:

The primary method is `React.memo()`, a built-in "higher order component" type. It accepts your own component type as an argument, and returns a new wrapper component. The wrapper component's default behavior is to check to see if any of the props have changed, and if not, prevent a re-render. Both function components and class components can be wrapped using `React.memo()`. (A custom comparison callback may be passed in, but it really can only compare the old and new props anyway, to the main use case for a custom compare callback would be only comparing specific props instead of all of them.)

The other options are:

- React.Component.shouldComponentUpdate:
  - an optional class component lifecycle method that will be called early in the render process. If it returns `false`, React will skip rendering the component. It may contain any logic you want to use to calculate that boolean result, but the most common approach is to check if the component's props and state have changed since last time, and return `false` if they're unchanged.
- React.PureComponent:
  - since that comparison of props and state is the most common way to implement `shouldComponentUpdate`, the `PureComponent` base class implements that behavior by default, and may be used instead of `Component` + `shouldComponentUpdate`.

All of these approaches uses a comparison technique called "shallow equality". This means checking every individual field in two different objects, and seeing if any of the contents of the objects are a different value. In other words, `obj1.a === obj2.a && obj2.a && obj1.b === obj2.b && .....`. This is typically a fast process, because `===` comparisons are very simple for the JS engine to do. So, these three approaches do the equivalent of `const shouldRender = !shallowEqual(newProps, prevProps)`.

There's also a lesser-known technique as well: if a React component returns the exact same element reference in its render output as it did the last time, React will skip re-rendering that particular child. There's at least a couple ways to implement this technique:

- If you include `props.children` in your output, that element is the same if this component does a state update
- If you wrap some elements with useMemo(), those will stay the same until the dependencies changes.

Examples:

![Ways to prevent rerenders](https://github.com/user-attachments/assets/8f129a62-9fbe-4d61-8f6e-977aac4c015c)

Conceptually, we could say that the difference between these two approaches is:

- `React.memo()`: controlled by the child component
- Same-element references: controlled by the parent component

For all of these techniques, skipping rendering a component means React will also skip rendering that entire subtree, because it's effectively putting a stop sign up to halt the default "render children recursively" behavior.
