## Keys and Reconciliation

The other way React identifies component "instances" is via the `key` pseudo-prop. React uses `key` as a unique identifier that it can use to differentiate specific instances of a component type.

Note that `key` isn't actually a real prop - it's an instruction to React. React will always strip that off, and it will never be passed through to the actual component, so you can never have `props.key` - it will always be undefined.

The main place we use keys is rendering lists. Keys are especially important here if you are rendering data that may be changed in some way, such as reordering, adding, or deleting list entries. It's particulary important here that keys should be some kind of unique IDs from your data if at all possible - only use array indices as keys as a last resort fallback!

![data object id as a key](https://github.com/user-attachments/assets/3c5467db-8991-405a-8c2c-a822aed6702a)

Here's an example of why this matters:

- Say I render a list of 10 <TodoListItem> components, using array indices as keys. React sees 10 items, with keys of `0..9`.
  - Now, if we delete items 6 and 7, and add three new entries at the end, we end up rendering items with keys of `0..10`.
  - So, it looks to React like I really just added one new entry at the end because we went from 10 list items to 11.
  - React will happily reuse the existing DOM nodes and component instances.
  - But, that means that we're probably now rendering <TodoListItem key={6}> with the todo item that was being passed to list item #8.
  - So, the component instance is still alive, but now it's getting a different data object as a prop than it was previously.
    - This may work, but it may also produce unexpected behavior.
    - Also, React will now have to go apply updates to several of the list items to change the text and other DOM contents, because the existing list items are now having to show different data than before.
    - Those updates really shouldn't be necessary here, since none of the list items changed.

If instead we were using `key={todo.id}` for each list item, React will correctly see that we deleted two items and added three new ones. It will destroy the two deleted component instances and their associated DOM, and create three component instances and their DOM. This is better than having to unnecessarily update the components that didn't actually change.

Keys are useful for component instance identity beyond lists as well. You can add a `key` to any React component at any time to indicate its identity, and changing that key will cause React to destroy the old component instance and DOM and create new ones. A common use case for this is a list + details form combination, where the form shows the data for the currently selected list item. Rendering <DetailForm key={selectedItem.id}> will cause React to destroy and re-create the form when the selected item changes, thus avoiding any issues with stale state inside the form.
