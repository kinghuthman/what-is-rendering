Direct copy/paraphased review of https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/#what-is-rendering

## Summary

- React always recursively renders components by default, so when a parent renders, its children will render

- Rendering by itself is fine - it's how React knows what DOM changes are needed

- But, rendering takes time, and "wasted renders" where the UI output didn't change can add up

- It's okay to pass down new references like callback functions and objects most of the time

- APIs like React.memo() can skip unnecessary renders if props haven't changed

- But if you always pass new references down as props, React.memo() can never skip a render, so you may need to memoize those values

- Context makes values accessible to any deeply nested component that is interested

- Context providers compare their value by reference to know if it's changed

- A new context values does force all nested consumers to re-render

- But, many times the child would have re-rendered anyway due to the normal parent->child render cascade process

- So you probably want to wrap the child of a context provider in React.memo(), or use {props.children}, so that the whole tree doesn't render all the time when you update the context value

- When a child component is rendered based on a new context value, React keeps cascading renders down from there too

- React-Redux uses subscriptions to the Redux store to check for updates, instead of passing store state values by context

- Those subscriptions run on every Redux store update, so they need to be as fast as possible

- React-Redux does a lot of work to ensure that only components whose data changed are forced to re-render

- connect acts like React.memo(), so having lots of connected components can minimize the total number of components that render at a time

- useSelector is a hook, so it can't stop renders caused by parent components. An app that only has useSelector everywhere should probably add React.memo() to some components to help avoid renders from cascading all the time.

- The "React Forget" auto-memoizing compiler may drastically simplify all this if it does get released.
