## How New Props References Affect Render Optimization

We've already seen that by default, React re-renders all nested components even if their props haven't changed. That also means that passing new references as props to a child component doesn't matter, because it will render whether or not you pass the same props. So, something like this is totally fine:

!(props affect render optimization)[https://github.com/user-attachments/assets/96c0e3f7-1fd1-460d-a345-a5bb68e6f28c]

Every time `ParentComponent` renders, it will create a new `onClick` function reference and a new `data` object reference, then pass them as props to `NormalChildComponent`. (Note that it doesn't matter whether we're defining `onClick` using the `function` keyword or as an arrow function - it
s a new function reference either way.)

However, if the child component is trying to optimize renders by checking to see whether props have changed, then passing new references as props will cause the child to render. If the new prop references are actually new data, this is good. However, what if the parent component is just passing down a callback function?

![component renders if unchanged props have a new reference](https://github.com/user-attachments/assets/9e00b813-389c-4501-8cd6-a207e3f39c7b)

Now ever timer `ParentComponent` renders, these new references are going to cause `MemoizedCHildComponent` to see that it's props values have changed to new references, and it will go ahead and re-render... even though the `onClick` function and the `data` object should be basically the same thing every time!

This means that:

- MemoizedChildComponent will always re-render even though we wanted to skip rendering most of the time
- The work that it's doing to compare its old and new props is wasted effort.

Similarly, note that rendering `<MemoizedChild><OtherComponent /><MemoizedChild>` will also force the child to always render, because `props.children` is always a new reference.
