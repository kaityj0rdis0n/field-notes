---
title: "Case styles: kebab, camel, Pascal, snake"
category: software
date: 2026-11-09
tags: [naming, fundamentals, css, javascript, react, conventions]
---

# Case styles: kebab, camel, Pascal, snake

## TL;DR

Code cannot have spaces inside a name, so programmers have a few conventions for writing multi-word names. **kebab-case** uses hyphens (`text-with-line`). **camelCase** capitalises every word after the first (`textWithLine`). **PascalCase** capitalises every word (`TextWithLine`). **snake_case** uses underscores (`text_with_line`). Which one you use is decided by where the name lives, not by preference.

## The explanation

### The four styles

| Style | Example | Memory hook |
|---|---|---|
| kebab-case | `text-with-line` | words skewered on a stick |
| camelCase | `textWithLine` | humps in the middle, flat at the start |
| PascalCase | `TextWithLine` | camelCase with a capital first letter |
| snake_case | `text_with_line` | words lying flat, joined by underscores |
| SCREAMING_SNAKE_CASE | `TEXT_WITH_LINE` | snake_case, shouting |

### Where each one lives

| Style | Used for | Why |
|---|---|---|
| kebab-case | CSS classes, CSS variables (`--color-text-primary`), HTML attributes, URLs, file names, string values | CSS and HTML are hyphen-friendly, and these names are never JavaScript identifiers |
| camelCase | JavaScript variables, functions, object keys, React prop *names* (`leadingIcon`) | JavaScript identifiers cannot contain hyphens |
| PascalCase | Classes, types, interfaces, React component names (`Button`, `PageHeading`) | React treats a lowercase tag as HTML and a capitalised one as a component |
| snake_case | Python, Ruby, SQL columns, environment-ish config | Those languages' conventions |
| SCREAMING_SNAKE_CASE | Constants and environment variables (`API_URL`) | Signals "this does not change" |

### The one that trips people up

A hyphen is the minus sign in JavaScript. So `text-with-line` cannot be a variable name: the parser reads it as `text` minus `with` minus `line`. It *can* be a string value, because strings can hold anything:

```tsx
// prop name: camelCase (it is an identifier)
// prop value: kebab-case (it is a string)
<Breadcrumbs type="text-with-line" leadingIcon={<ChevronIcon />} />
```

This is why design-system components end up with camelCase prop names and kebab-case prop values in the same line. It is not inconsistency. Each name is following the rules of the place it lives.

### The React capitalisation rule

In JSX, `<button>` and `<Button>` are different things. Lowercase is an HTML element. Capitalised is a component you wrote or imported. This is why component names are always PascalCase: a lowercase component name would silently render as an unknown HTML tag.

## When does this matter?

- **Reading unfamiliar code.** The casing tells you what kind of thing a name is before you look it up. `PageHeading` is a component. `pageHeading` is a variable. `page-heading` is a CSS class or a string.
- **Translating design labels into code.** A Figma option "Text with line" becomes the string `"text-with-line"`. See [What's a prop value?](prop-values.md).
- **Naming files.** Markdown and config files are usually kebab-case (`fast-forward.md`). React component files often match the component: `Button.tsx`.
- **Searching.** If a grep for `textWithLine` finds nothing, try `text-with-line` and `TextWithLine`. The same concept can appear in three casings across CSS, JavaScript and types.

## See also

- [What's a prop value?](prop-values.md): where kebab-case values and camelCase names meet
- [Property shorthand & exact-match lookup (JS)](property-shorthand.md): another case where "means the same" is not "is the same name"
- [MDN: JavaScript identifiers](https://developer.mozilla.org/en-US/docs/Glossary/Identifier)
- [React docs: naming components](https://react.dev/learn/your-first-component#step-2-define-the-function): why component names start with a capital letter
