## Immutability and Rerendering

State updates in React should always be done immutably. There are two main reasons why:

- depending on what you mutate and where, it can result in component not rendering when you expected they would render
- it causes confusion about when and why data actually got updated

Let's look at a couple specific examples.

As we've seen, `React.memo / PureComponent / shouldComponentUpdate` all rely on shallow equality checks of the current props vs the previous props. So, the expectation is that we can know if a prop is a new value, by doing `props.someValue !== prevProps.someValue`.

If you mutate, then `someValue` is the same reference, and those components will assume nothing has changed.

Note that this is specifically when we're trying to optimize performance by avoiding unnecessary re-renders. A render is "unnecessary" or "wasted" if the props haven't changed. If you mutate, the component may wrongly think nothing has changed, and then you wonder why the component didn't re-render.

The other issue is the `useState` and `useReducer` hooks. Every time I call `setCounter()` or `dispatch()`, React will queue up a re-render. However, React requires that any hook state updates must pass in / return a new reference as the new state value, whether it be a new object/array reference, or a new primitive (string/number/etc.)

React applies all state updates during the render phase. When React tries to apply a state update from a hook, it checks to see if the new value is the same reference. React will always finish rendering the component that queued the update. However, if the value is the same reference as before, and there are no other reasons to continue rendering (such as the parent having rendered), React will then throw away the render results for component and bail out of the render pass completely and bail out of the render pass completely. So, if I mutate an array like this:

![improperly update state object](https://github.com/user-attachments/assets/594b51aa-99e0-4ccc-800b-b63888a084f9)

then the component will fail to re-render.

(Note that React does actually have a "fast path" bailout mechanism that will attempt to check the new value before queuing the state update in some cases. Since this also relies on a direct reference check, it's another example of needing to do immutable updates.)

Technically, only the outermost reference has to be immutably updated. if we change that example to:

![update outermost reference immutably](https://github.com/user-attachments/assets/8e34acaa-857c-4e35-9665-a74e2361ced6)

then we have created a new array reference and passed it in, and the component will re-render.

Note that there is a distinct difference in behavior between the class component `this.setState()` and the function component `useState` and `useReducer` hooks with regards to mutations and re-rendering. `this.setState() doesn't care if you mutate at all - it always completes the re-render. So, this will re-render:

![class component setState always completes the rerender](https://github.com/user-attachments/assets/5e3e21ee-42c1-4fe9-b166-3b4a19a666d8)

And in fact, so will passing in an empty object like `this.setState({})`.

Beyond all the actual rendering behavior, mutation introduces confusion to the standard React one-way data flow. Mutation can lead other code to see different values when the expectation was they haven't changed at all. This makes it harder to know when and why a given piece of state was actually supposed to update, or where a change came from.

Bottom line: React, and the rest of the React ecosystem, assume immutable updates. Any time you mutate, you run the risk of bugs.
