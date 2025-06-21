## Render Behavior Edge Cases

### Commit Phase Lifecycle

There's some additional edge cases inside of the commit-phase lifecycle methods: `componentDidMount`, `componentDidUpdate` and `useLayoutEffect`. These largely exist to allow you to perform additional logic after a render, but before the browser has had a chance to paint. In particular, a common use case is:

- Render a component the first time with some partial but incomplete data
- In a commit-phase lifecycle, use refs to measure the real size of the actual DOM nodes in the page
- Set some state in the component based on those measurements
- Immediately re-render with the updated data

In this use case, we don't want the initial "partial" rendered UI to be visible to the user at all - we only want the "final" UI to show up. Browsers will recalculate the DOM structured as it's being modified, but they won't actually paint anything to the screen while a JS script is still executing and blocking the event loop. So, you can perform multiple DOM mutations, like `div.innerHTML = "a"; div.innerHTML = b";`, and the `"a"` will never appear.

Because of this, react will always run renders in commit-phase life-cycles synchronously. That way, if you do try to perform an update like that "partial->final" switch, only the "final" content will ever be visible on screen.

As far as I know, state updates in `useEffect` callbacks are queued up, and flushed at the end of the "Passive Effects" phase once all the `useEffect` callbacks have completed.

### Reconciler Batching Methods

React Reconcilers (ReactDOM, React Native) have methods to alter render batching.

For React 17 and earlier, you can wrap multiple updates that are outside of event handlers in `unstable_batchedUpdates()` to batch them together. (Note that despite the `unstable_` prefix, it's heavily used and depended on by code at Facebook and public libraries - [React-Redux v7 used `unstable_batchedUpdates` internally](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/#use-of-react-s-batched-updates-api)).

Since React 18 automatically batches by default, React 18 has a [flushSync() API](https://react.dev/reference/react-dom/flushSync) that you can use to force immediate renders and opt out of automatic batching.

Note that since these are reconciler-specific APIs, alternate reconcilers like `react-three-fiber` and `ink` may not have them exposed. Check the API declarations or implementation details to see what's available.

#### <StrictMode>

React will double-render components inside of a `<StrictMode>` tag in development. That means the number of times your rendering logic is not the same as the number of committed render passes, and you cannot rely on `console.log()` statements while rendering to count the number of renders that have occurred. Instead, either use the React DevTools Profiler to capture a trace and count the number of committed renders overall, or add logging inside of a `useEffect` hook or `componentDidMount/Update` lifecycle. That way the logs will only get printed when React has actually completed a render pass and committed it.

### Setting State While Rendering

In normal situations, you should never queue a state update while in the actual rendering logic. In other words, it's fine to create a click callback that will call `setSomeState()` when the click happens, but you should not call `setSomeState() as part of the actual rendering behavior.

However, there is one exception to this. Function components may call `setSomeState()` directly while rendering, as long as it's done conditionally and isn't going to execute every time this component renders. This acts as the function component equivalent of [getDerivedStateFromProps in class components](https://legacy.reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops). If a function component queues a state update while rendering, [React will immediately apply the state update and synchronously re-render that one component before moving onwards](https://github.com/facebook/react/blob/v18.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L430-L469). If the component infinitely keeps queuing state updates and forcing React to re-render it, React will break the loop after a set number of retries and throw and error (currently 50 attempts). This technique can be used to immediately force an update to a state value based on a prop change, without requiring a re-render + a call to `setSomeState()` inside of a `useEffect`.
