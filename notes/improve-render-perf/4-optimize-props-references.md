## Optimizing Props References.

Class components don't have to worry about accidentally creating new callback function references as much, because they can have instance methods that are always the same reference. However, they may need to generate unique callbacks for separate child list items, or capture a value in anonymous function and pass that to a child. Those will result in new references, and so will creating new objects as child props while rendering. React doesn't have anything built-in to help optimize those cases.

For function components, React does provide two hooks to help you reuse the same references: `useMemo` for any kind of general data like creating objects or doing complex calculations, and `useCallback` specifically for creating callback functions.
