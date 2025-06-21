## Async Rendering, Closures, and State Snapshots

One extremely common mistake we all the time is when a user sets a new value, then tries to log the existing variable name. However, the original value gets logged, not the updated value.

![mistake logging state value after setting](https://github.com/user-attachments/assets/d79f77db-fcca-4e6a-8a6d-3e1d4924550e)

Why doesn't this work?

As mentioned above, it's common for experienced users to say "React state updates are async". This is sort of true, but there's a bit more nuance then that, and actually a couple different problems at work here.

Strictly speaking, the React render is literally synchronous - it will be executed in a "microtask" at the very end of this event loop tick. (This is admittedly being pedantic, but the goal of this "article" is exact details and clarity.) However, yes, from the point of view of that `handleClick` function, it's "async" in that you can't immediately see the results, and the actual update occurs much later than the `setCounter()` call.

However, there's a bigger reason why this doesn't work. The `handleClick` was defined during the most recent render of this function component, it can only see the value of variables as they exited when the function was defined. In other words, these state variables are a snapshot in time.

Since `handleClick` was defined during the most recent render of this function component. it can only see the value of `counter` as it existed during that render pass. When we call setCounter(), it queues up a future render pass, and that future render will have a new `counter` variable with the new value and a new `handleClick` function... but this copy of `handleClick` will never be able to see that new value.

The new React docs cover this in more detail in the section [State as a Snapshot](https://react.dev/learn/state-as-a-snapshot), which is highly recommended reading.

Going back to the original example: trying to use a variable right after you set an updated value is almost always the wrong approach, and suggests you need to rethink how you are trying to use that value.
