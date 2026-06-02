---
parent: Development
grand_parent: VAYS
nav_order: 2
---

# React Primer

Background reading for maintainers who haven't worked with React
before. If you already know React, skip to the
[JSON Forms primer](json-forms.md) — that's the part of the stack
that's actually unusual.

## Getting Started

You'll need Node.js (and `npm`). I personally recommend
[`nvm`](https://github.com/nvm-sh/nvm) for installing both.

The fastest way to learn React is to build something throw-away. Grab
the official starter, clone it, and add a few components of your own
(`npx` is just the "run a package" shortcut):

```sh
npx create-react-app my-app --template typescript
cd my-app
```

Further reading:

  - **Master React in 5 Days** — downloadable free of charge inside
    the ETH network: <https://link.springer.com/book/10.1007/978-1-4842-9855-8>.
  - The official tutorial at <https://react.dev/learn>.

## Common Pitfalls

### Components return a single element

A component must return exactly one element. If you have several
siblings, wrap them in a fragment:

```jsx
const Component = () => (
  <>
    <div id="div1" />
    <div id="div2" />
  </>
);
```

### Hooks live inside components

`useState`, `useEffect`, etc. may only be called inside a component
(or inside a custom hook). Calling them from a regular helper
function will throw at runtime.

### Functional vs. class components

You'll see both styles in older React code. Class components are
legacy; functional components plus hooks are the current standard and
generally perform better. The one caveat: a functional component has
no instance methods, which can matter for singleton-like UI. VAYS uses
a class component for exactly one thing — the modal — because we want
to call `Modal.show(...)` from anywhere.

### Performance

When a parent rerenders, its children rerender by default — even if
their props haven't changed. For expensive components, wrap them in
[`React.memo`](https://react.dev/reference/react/memo) so they only
rerender when their props actually change.
