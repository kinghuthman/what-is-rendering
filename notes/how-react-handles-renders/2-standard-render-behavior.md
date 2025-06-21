## Standard Render Behavior

React's default behavior, when a parent component renders, its child components will recursively render too

As an example: A > B > C > D

- The user clicks a button that increments a counter in component B
  - we call setState in component B which requeues a rerender of B
    - React starts the render past from the top of the tree
      - React sees that A is not in need of an update so passes on
        - React sees that B is in need of an update and renders it. B returns <C /> as it did last time
          - C was not marked as needing an update but because it's a parent B did, it will now re-render as React moves downward. C returns <D />
            - D was also not marked for rendering but because C did, react moves downward and re-renders D too

To keep it simple, re-rendering a component re-renders all components within it as well. It doesn't care if props change, if the parent re-renders, it will re-render the child as well.

This means that calling setState() in your root <App> component will cause every component in the component tree to re-render.

Rendering is not a bad thing - it's how react knows whether it actually needs to make changes to the DOM

## Rules of React Rendering

One of the rules of react rendering is that it must be "pure" and not have any side effects!

- This can be tricky and confusing, because many side effects don't break anything.
  - Strictly speaking a console.log() is a side effect
  - Mutating a prop is a side effect and it might not break anything
  - Making an AJAX call in the middle of rerendering is definitely a side effect and can definitely cause unexpected app behavior
