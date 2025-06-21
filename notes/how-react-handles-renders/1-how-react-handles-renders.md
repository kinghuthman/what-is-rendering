# How does react handle renders

## Queue Renders

After the initial render is completed, react has a few different ways to queue re-renders

- Functional Components
  - useState setters
  - useReducer dispatches
- Class Components
  - this.setState()
  - this.forceUpdate()
- Other
  - calling the ReactDom top level render(<App>) method again which is similar to forceUpdate() on the root component

Note that function components don't have a force update method, but you can get the same behavior by using a useReducer hook that always increments a counter: `const [, forceRender] = useReducer((c) c+ 1, 0)`

### Sebastian Markbage "Rules of React"

He defines the expected behavior for react lifecycle methods, including render. https://gist.github.com/sebmarkbage/75f0838967cd003cd7f9ab938eb1958f

- Render logic must not:

  - can't mutate existing variables and objects
  - can't create random values like Math.random() or Date.now()
  - can't make network requests
  - can't queue state updates

- Render logic may:
  - mutate objects that were newly created while rendering
  - throw errors
  - "lazy initialize" data that hasn't been created yet, such as a cached value
