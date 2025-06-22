## Updating Context Values

React checks to see if a context provider has been given a new value when the surrounding component renders the provider. If the provider's value is a new reference, then React knows the value has changed, and that the components consuming that context need to be updated.

Note that passing a new object to a context provider will cause it to update:

![new object updates context provider](https://github.com/user-attachments/assets/c702c5b8-8e21-43af-9eb3-01df98d14a6b)

In this example, every time `ParentComponent` renders, React will take note that `MyContext.Provider` has been given a new value, and look for components that consume `MyContext` as it continues looping downwards. When a context provider has a new value, every nested component that consumes that context will be forced to re-render.

Note that from React's perspective, each context provider only has a single value - doesn't matter whether that's an object, array, or a primitive, it's just one context value. Currently, there is no way for a component that consumes a context to skip updates caused by new context values, even if it only cares about part of a new value.

If a component only needs `value.a`, and an update is made to cause a new `value.b` reference... the rules of immutable updates and context rendering require that value be a new reference also, and so the component reading `value.a` will also render too.

## State Updates, Context, and Re-Renders

It's time to put some of these pieces together. We know that:

- Calling `setState()` queues a render of that component
- React recursively renders nested components by default
- Context providers are given a value by the component that renders them
- That value normally comes from that parent component's state

This means that by default, any state update to a parent component that renders a context provider will cause all of its descendants to re-render anyway, regardless of whether they read the context value or not!.

If we look back at the `Parent/Child/Grandchild` example just above, we can see that the `GrandchildComponent` will re-render, but not because of a context update - it will re-render because ChildComponent rendered!. In this example, there's nothing trying to optimize away "unnecessary" renders, so React renders `ChildComponent` and `GrandchildComponent` by default any time `ParentComponent` renders. If the parent puts a new context value into `MyContext.Provider`, the `GrandchildComponent` will see the new value when it renders and use it, but the context update didn't cause `GrandchildComponent` to render - it was going to happen anyway.

## Context Updates and Render Optimizations

Let's modify that example so that it does actually try to optimize things, but we'll add one other twist by putting a `GreatGrandchildComponent` at the bottom:

![Optimize components under parents that consume context](https://github.com/user-attachments/assets/89fb6fab-b347-4366-90c7-7d6f9c81a9ed)

Now, if we call `setA(42)`:

- `ParentComponent` will render
- A new contextValue reference is created
- React sees that `MyContext.Provider` has a new context value, and thus any consumers of `MyContext` need to be updated
- React will try to render `MemoizedChildComponent`, but see that it's wrapped in `React.memo()`. There are no props being passed at all, so the props have not actually changed. React will skip rendering `ChildComponent` entirely.
- However, there was an update to `MyContext.Provider`, so there may be components further down that need to know about that.
- React continues downwards and reaches `GrandchildComponent`. It sees that `MyContext` is read by `GrandchildComponent`, and thus it should re-render because there's a new context value. React goes ahead and re-renders GrandchildComponent, specifically because of the context change.
- Because `GrandchildComponent` did render, React then keeps on going and also renders whatever's inside of it. So, React will also re-render `GreatGrandchildComponent`.

["That React Component Right Under Your Context Provider Should Probably Use React.memo" - Sophie Alpert](https://twitter.com/sophiebits/status/1228942768543686656)

That way, state updates in the parent component will not force every component to re-render, just the sections where the context is read. (You could also get basically the same result by having `ParentComponent` render `<MyContext.Provider>{props.children}</MyContext.Provider>`, which leverages the "same element reference" technique to avoid child components re-rendering, and then rendering `<ParentComponent><ChildComponent /></ParentComponent>` from one level up.)

Note, however, that once `GrandchildComponent` rendered based on the next context value, React went right back to its default behavior of recursively re-rendering everything. So, `GreatGrandchildComponent` was rendered, and anything else under there would have rendered too.
