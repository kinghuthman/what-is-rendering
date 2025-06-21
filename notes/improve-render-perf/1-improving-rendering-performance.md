# Improving Rendering Performance

Although renders are the normal expected part of how React works, it's also true that render work can be "wasted" effort at times. If a component's render output didn't change, and that part of the DOM didn't need to be updated, then the work of rendering that component was really kind of a waste of time.

React component render output should always be entirely based on current props and current component state. Therefore, if we know ahead of time that a component's props and state haven't changed, we should also that the render output would be the same, that no changes are necessary for this component, and that we can safely skipthe work of rendering it.

When trying to improve software performance in general, there are two basic approaches: 1) do the same work faster, and 2) do less work. Optimizing React rendering is primarily about doing less work by skipping rendering components when appropriate.
