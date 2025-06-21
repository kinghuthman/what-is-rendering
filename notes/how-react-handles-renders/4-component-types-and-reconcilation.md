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
