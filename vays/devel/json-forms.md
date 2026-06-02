---
parent: Development
grand_parent: VAYS
nav_order: 3
---

# JSON Forms Primer

[JSON Forms](https://jsonforms.io) renders the parameters declared in
the JSON Schema that YAC returns from its `/validate` endpoint. It's
framework-agnostic; we use the React bindings.

## The Top-Level Component

The whole form is a single component:

```jsx
<JsonForms
  schema={jsonSchema}
  uischema={uiSchema}
  data={data}
  renderers={renderers}
  cells={materialCells}
  onChange={({ errors, data }) => { /* ... */ }}
/>
```

  - `schema` — the JSON Schema returned by YAC.
  - `uischema` — the UI Schema returned by YAC.
  - `data` — values for the parameters declared in the schema. Not
    every parameter has to be present.
  - `renderers` — an array of `{ renderer, tester }` pairs (see
    below).
  - `cells` — smaller inline renderers. **Not used in this project.**
  - `onChange` — fires when the data changes. Renderers have some
    control over *when* this fires, which matters for performance —
    see [Debouncing](#performance-trap-debounce-text-input) below.

## Writing Renderers

A renderer is a `{ renderer, tester }` pair registered with JSON
Forms. The **renderer** is a React component that takes the JSON
Forms props and returns JSX. The **tester** is a function that
inspects the current UI-Schema subtree plus the resolved JSON-Schema
and returns a numeric rank if it can render the field. Highest rank
wins; ties break by registration order.

### Basic Tester

```jsx
export const TextControlTester: RankedTester = rankWith(
  2,
  isStringControl,
);
```

### Basic Renderer

A trimmed-down version of `TextControl` from the codebase:

```jsx
export const TextControl = (props: ControlProps) => {
  const eventToValue = (ev: any) =>
    ev.target.value === '' ? undefined : ev.target.value;

  const onChange = useCallback(
    debounce(
      (e: React.ChangeEvent<HTMLInputElement>) =>
        props.handleChange(props.path, eventToValue(e)),
      1500,
    ),
    [props.path],
  );

  return <TextInput {...props} onChange={onChange} onClear={onClear} />;
};
```

## Further Reading

  - JSON Forms custom-renderers tutorial: <https://jsonforms.io/docs/tutorial/custom-renderers>.
  - Passing custom props: <https://jsonforms.discourse.group/t/how-do-i-pass-custom-props/2110/7>.
  - Performance / debouncing: <https://jsonforms.discourse.group/t/custom-renderer-tester-docs-are-lacking/204>.
  - Core interfaces: <https://jsonforms.io/api/core/interfaces/>.

## Common Pitfalls

### Importing the unwrapped control

When you import a control, **don't** destructure it out of its module
without also keeping the default import:

```jsx
import TextControl, { TextControlTester } from './TextControlRenderer';
```

The default export is wrapped with `withJsonFormsControlProps(...)`,
which is what injects props like `description`. Importing only the
named binding bypasses the wrapper and the control will misbehave.

### `useState` inside a renderer

Avoid `useState` and the corresponding setters inside custom control
components. Calling a setter queues a rerender, which can fight with
JSON Forms' own rerender management; the typical result is an infinite
loop that JSON Forms bails out of by throwing.

### Performance trap: debounce text input

The whole form revalidates on every `onChange` — so an unthrottled
text input revalidates the entire JSON Schema on every keystroke.
Past a certain schema size, the UI stops being typeable. Debounce the
handler (see `TextControl` above) so revalidation happens at most
every ~1.5 seconds.

### Other performance tips

`React.memo` on heavy subtrees helps for the same reason it does
elsewhere — see the [React primer](react.md#performance).
