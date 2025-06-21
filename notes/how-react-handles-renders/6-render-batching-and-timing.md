## Render Batching and Timing

By Default, each call to `setState()` causes React to start a new render pass, execute it synchronously, and return. However, React also applies a sort of optimization automatically, in the form of render batching. Render batching is when multiple calls to `setState()` result in a single render pass being queued and executed, usually on a slight delay.

The React community often describes this as "state updates may be asynchronous". The new React docs also describe it as "State is a Snapshot". That's a reference to this render batching behavior.

In react 17 and earlier, React only did batching in React event handlers such as `onClick` callbacks. Updates queued outside of event handlers, such as `setTimeout`, after an `await`, or in a plain JS event handler, were not queued, and would each result in a separate re-render.

However, [React 18 now does "automatic batching" of all updates queued in any single event loop tick.](https://github.com/reactwg/react-18/discussions/21) This helps cut down on the overall number of renders needed.

Example

![render batching before react 18](https://github.com/user-attachments/assets/e3511e1e-e4e4-4d54-b5a9-bdff7ba27941)

With React 17, this executed three render passes. The first pass will batch together `setCounter(0)` and `setCounter(1)`, because both of them are occurring during the original event handler call stack, and so they're both occurring inside the `unstable_batchedUpdates()` call. However, the call to `setCounter(2)` is happening after an `await`. This means the original synchronous call stack is done, and the second half of the function is running much later in a totally separate event loop call stack. Because of that, React will execute an entire render pass synchronously as the last step inside the `setCounter(2)` call, finish the pass, and return from `setCounter(2)`.

The same thing will then happen for `setCounter(3)`, because it's also running outside the original event handler, and thus outside the batching.

However, with React 18, this executes two render passes. The first two, setCounter(0) and setCounter(1), are batched together because they're in one event loop tick. Later, after the await, both `setCounter(2)` and `setCounter(3)` are batched together - even though they're much later, that's also two state updates queued in the same event loop, so they get batched into a second render.
