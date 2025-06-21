## Memoize Everything?

As mentioned above, you don't have to throw `useMemo` and `useCallback` at every single function or object you pass down as a prop - only if it's going to make a difference in behavior for the child. (That said, the dependency array comparisons for `useEffect` do add another use case where the child might want to receive consistent props references, which does make things more complicated.)

The other question that comes up all the time is "Why doesn't React wrap everything in `React.memo()` by default?".

Dan Abramov has repeatedly pointed out that memoization does still incue the cost of comparing props, and that there are many caes whwere the memoization check can never prevent re-renders because the component always receives new props. As an example, [see this Twitter/X thread from Dan](https://twitter.com/dan_abramov/status/1083897065263034368):

[Thread between mark and dan](https://twitter.com/acemarke/status/1141755698948165632)

[When not to use React.memo](https://github.com/facebook/react/issues/14463)

[React docs, should you memo everywhere?](https://react.dev/reference/react/memo#should-you-add-memo-everywhere)
