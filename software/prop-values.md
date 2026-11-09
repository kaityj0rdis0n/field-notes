---
title: What's a prop value?
category: software
date: 2026-11-09
tags: [react, javascript, components, props, fundamentals, typescript]
---

# What's a prop value?

## TL;DR

A **prop** is a named setting a component accepts. A **prop value** is what you actually put in that setting when you use the component. The prop is the slot; the value is what fills it. `size` is a prop. `"lg"` is a prop value.

## The explanation

### Slot and filling

Think of a form with labelled boxes. The labels are the props. What you write in the boxes are the prop values.

```tsx
<Button size="lg" disabled={false} onClick={save}>Save</Button>
```

| Prop (the slot) | Prop value (what fills it) |
|---|---|
| `size` | `"lg"` |
| `disabled` | `false` |
| `onClick` | the `save` function |
| `children` | the text `Save` |

The component's author decided which slots exist. The person using the component decides what goes in them, within whatever the author allows.

(For how props work as function arguments, see [What's a prop?](react-props.md). This note is about the value side.)

### A prop value can be any kind of data

Props are not limited to text. Anything JavaScript can hold can be a prop value:

| Kind | Example | Written in JSX as |
|---|---|---|
| String | `"lg"` | `size="lg"` |
| Number | `3` | `count={3}` |
| Boolean | `true` / `false` | `disabled={true}`, or just `disabled` for true |
| Function | `save` | `onClick={save}` |
| Object or array | `{ id: 1, name: "Em" }` | `user={someUser}` |
| Another component | `<ChevronIcon />` | `leadingIcon={<ChevronIcon />}` |
| Nothing | `undefined` | leave the prop off |

Two syntax rules fall out of this. Plain quotes `"..."` always mean a string. Curly braces `{...}` mean "evaluate this JavaScript and use the result," which is how numbers, booleans, functions and variables get in.

### Where the value comes from

The value does not have to be typed in by hand. It can come from anywhere the code has access to:

```tsx
// hardcoded
<Button size="lg" />

// from a variable
const buttonSize = isMobile ? "sm" : "lg";
<Button size={buttonSize} />

// from state, so it changes while the app runs
const [saving, setSaving] = useState(false);
<Button loading={saving} />

// from the parent's own props, passed straight through
function Toolbar({ size }) {
  return <Button size={size} />;
}
```

In the last case the same value travels down two levels. That chain is what [prop drilling](prop-drilling.md) describes.

### The author decides which values are allowed

The component's TypeScript interface says what each prop accepts. For many props that is a short list of allowed strings:

```tsx
interface ButtonProps {
  size?: "xs" | "sm" | "md" | "lg" | "xl";
  disabled?: boolean;
  onClick?: () => void;
}
```

`"xs" | "sm" | "md" | "lg" | "xl"` is a **string literal union**: a type that means "one of exactly these strings." Pass `size="huge"` and TypeScript refuses to compile. That is the prop value being checked against the slot's shape.

The `?` means the prop is optional. When it is left off, the value is `undefined` and the component usually falls back to a default:

```tsx
function Button({ size = "md", ...rest }) { ... }
```

Here `"md"` is the **default prop value**: what the slot holds when nobody fills it.

### Value vs identity

Two renders with the same prop values look the same. The component does not care where the value came from, only what it is. `size="lg"` typed by hand and `size={buttonSize}` where `buttonSize` happens to be `"lg"` produce identical output. This is why components are reusable: they respond to values, not to who supplied them.

## When does this matter?

- **Reading a component's usage.** Everything between `<` and `>` after the name is a prop-value pair. The left side tells you the slot; the right side tells you what this particular use chose.
- **Something renders wrong.** Check the value that reached the component, not just the prop name. A prop can be spelled right and still hold the wrong value, especially when it came from state or a variable several layers up.
- **TypeScript errors on a component.** The message usually says a value is "not assignable" to the prop's type. That is the allowed-values list rejecting what you passed.
- **Optional props.** If a component behaves differently when you omit a prop, the default value is doing it. Look for `= something` in the component's destructuring.
- **Design systems.** When a component is built from a design tool, the allowed string values for a prop are expected to mirror the design tool's variant options, so `"text-with-line"` in code and "Text with line" in Figma are the same thing. Lowercase the label, swap spaces for hyphens, and one search finds both the code and the frame.

## See also

- [What's a prop? (React)](react-props.md): props as function arguments
- [Prop drilling](prop-drilling.md): what happens when a value has to travel through several components
- [Function signatures](function-signatures.md): the broader idea of a declared contract for inputs
- [Case styles](naming-conventions-case-styles.md): why prop names are camelCase and string values are often kebab-case
- [React docs: passing props to a component](https://react.dev/learn/passing-props-to-a-component)
- [TypeScript handbook: literal types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types): the "one of exactly these strings" mechanism
