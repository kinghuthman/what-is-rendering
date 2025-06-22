# Context and Rendering Behavior

React's Context API is a mechanism for making a single user-provided value available to a subtree of components, Any component inside of a given `<MyContext.Provider>` can read the value from that context instance, without having to explicitly pass that value as a prop through every intervening component.

Context is not a "state management" tool. You have to manage the values that are passed into context yourself. This is typically done byb keeping data in React component state, and constructing context values based on that data.

## Context Basics

A context provider receives a single value prop, like `<MyContext.Provider value={42}>`. Child components may consume the context by rendering the context consumer component and providing a render prop, like:

`<MyContext.Consumer>{ (value) => <div>{value}</div>}</MyContext.Consumer>`

or by calling the useContext hook in a function component:

`const value = useContext(MyContext)`
