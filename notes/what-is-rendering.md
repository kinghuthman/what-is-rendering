# What is "Rendering"?

Rendering is the process of React asking your components to describe what they want their section of the UI to look like, now, based on the current combination of props and state.

## Rendering Process Overview

During the rendering process, react starts at the root of the component tree and loops downwards to find all components that have been flagged as needing updates.

- What method is used to loop downward?
- What flagging is used to determine a component needs an update

For each flagged component, react will call either FunctionComponent(props) (for function components), or classCOmponentInstance.render() (for class components), and save the render output for the enxt steps of the render pass.

A component's render output is normally written in JSX syntax, which is then converted to React.createElement() calls as the JS is compiled and prepared for deployment.
createElement returns React elements, which are plan JS objects that describe the intended structure of the UI.

### For example:

- This syntax
  - <MyComponent a={42} b="testing">Text Here</MyComponent>
    - is converted to this call:
      - return React.createElement(MyComponent, {a:42, b: "testing"}, "Text Here")
        - and that becomes this element object:
          - {type: MyComponent, props: {a: 42, b: "testing"}, children: ["Text Here"]}
            - and internally, React calls the actual function to render it:
              - let elements = MyComponent({...props, children})
- For "host components" like HTML:
  - return <button onClick={() =>{}}>Click Me</button>
    - becomes
      - return React.createElement("button", {onClick}, "Click Me")
        - and finally
          - {type: "button", props: {onClick}, children: ["Click me"]}

After it has collected the render output from the entire component tree. React will diff the new tree of objects(frequently referred to as the "virtual dom"), and collects a list of all the changes that need to be applied to make the real DOM look like the current desired output. The diffing and calculation process is known as "reconciliation".

- How does react diff the new tree?
- How does it collect a list of that changes that need to be made
- Reconciliation:
  - the action of making one view or belief compatible with another
  - the restoration of friendly relations

React then applies all the calculated changes to the DOM in one synchronous sequence.

- React has downplayed "virtual dom"
  - "React is value UI. Its core princple is that UI is a value, just like a string or an array. You can keep it in a variable, pass it around, use javaScript control flow with it, and so on. That expressiveness is the point -- not some diffing to avoid applying changes to the DOM. It doesn't even always represent the DOM"
    - For example:
      - <Message recipientId={10} />
          - is not DOM. Conceptually it represents lazy function calls:
              - Message.bind(null, {recipientId: 10})

## Render and Commit Phases

This is divided into two phases, conceptually:

- - The "Render phase" contains all the work of rendering components and calculating changes
  - The "commit phase" is the process of applying those changes to the DOM

After React has updated the DOM in the commit phase, it updates all refs accordingly to point to the requested DOM nodes and component instances. It then synchronously runs the componentDidMount and componentDidUpdate class lifecycle methods, and the useLayoutEffect hooks.

- How do the refs point to the correct nodes and component instances?
- synchronous:
  - existing or occurring at the same time

React then sets a short timeout, and when it expires, runs all the useEffect hooks. This step is also known as the "Passive Effects" phase.

- How long is the timeout
- At the same time does it run all the useEffect hooks or in a specific or random order?
  - how is that specific order determined?

React 18 added "Concurrent Rendering" features like useTransition. This gives React the ability to pause the work in the rendering phase to allow the browser to process events.

- concurrent:
  - existing, happening or done at the same time

This gives react the ability to pause the work in the rendering phase to allow the browser to process events. React will either resume, throw away, or recalculate that work later as appropriate. Once the render pass has been completed, React will run the commit phase synchronously in one step.

- How does it determine to "resume, throw away or recalculate work later as appropriate"?

A key part of this to understand is that "rendering" is not the same thing as "updating the DOM", and a component may be rendered without any visible changes happening as a result. When React renders a component:

- The component might return the same render output as last time, so no changes are needed
- In Concurrent Rendering, React might end up rendering a component multiple times, but throw away the render output each time if other updated invalidate the current work being done

https://julesblom.com/writing/react-hook-component-timeline
https://wavez.github.io/react-hooks-lifecycle/

- Note... if a component returns child components, the effects are called from the children upwards to the parent.
