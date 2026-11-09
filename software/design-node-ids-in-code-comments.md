---
title: Design node IDs in code comments
category: software
date: 2026-11-09
tags: [design-systems, figma, traceability, code-comments, design-to-code]
---

# Design node IDs in code comments

## TL;DR

Every layer in a Figma file has an ID like `1234:5678`. Putting that ID in a code comment next to the code built from it means anyone can jump from the code to the exact drawing, or from the drawing to the exact code. Without it, matching code to design is guesswork.

## The explanation

### What a node ID is

Figma treats every frame, component, group and layer as a **node** in a tree, and gives each one an ID: two numbers separated by a colon, like `1234:5678`.

When you select something in Figma and choose **Copy link**, the ID is in the URL:

```
https://www.figma.com/design/<fileKey>/<fileName>?node-id=1234-5678
```

Note the hyphen in the URL and the colon everywhere else. Converting between them is the only translation step.

### What a code comment is

A comment is text inside a source file that the computer ignores and a person reads. In JavaScript and TypeScript:

```tsx
// this is a comment
```

### Putting the two together

A design-system component built from a Figma frame carries the frame's ID in a comment at the point where the code matches the drawing:

```tsx
// Destructive variants: match Figma node 1234:5678
{
  destructive: true,
  hierarchy: "primary",
  className: "bg-[var(--color-bg-error-solid)] text-white",
},
```

```tsx
// Spinner arc starts at 12 o'clock and sweeps to 3 o'clock.
// Matches the "Line" layer in Figma node 1234:9012.
<path d="M12 2a10 10 0 0 1 10 10" />
```

Some teams also emit the ID as a `data-node-id` attribute on the rendered HTML, so the mapping survives into the browser's inspector.

### Why it is worth the clutter

Traceability runs both ways.

- **Code to design.** An engineer reading an odd-looking line pastes the ID into the Figma URL and lands on the drawing that justifies it. No "why is this 544px?" thread.
- **Design to code.** A designer changes a frame, searches the codebase for its ID, and finds every piece of code that now needs to follow. No "did anyone update the modal?" thread.
- **Review.** A reviewer can open the frame and the code side by side and check them against each other, instead of trusting the PR description.

The automated version of this idea is Figma Code Connect, which maps a Figma component to its code so Dev Mode shows the real snippet. Node IDs in comments are the manual, zero-tooling version. They work on any Figma plan and survive a file move as long as the IDs are preserved.

### What it does not do

The ID points at a location, not a version. If the frame changes, the comment still points at it, but the code may no longer match. The comment tells you *where* to look, not *whether* it is still right. Drift detection is a separate job.

## When does this matter?

- **Building a component from a design.** Record the node ID per variant as you go. It costs one line and saves the archaeology later.
- **A design changed and you own the code.** Search for the ID first. If nothing comes up, the component was built without traceability and needs a visual comparison instead.
- **Reviewing design-system PRs.** A component with no node IDs in it was probably built from memory or from a neighbouring component. Ask where the design is.
- **Writing a component request.** Paste the Figma link. The node ID in it is the single most useful thing you can hand an engineer.

## See also

- [What's a prop value?](prop-values.md): the values a component accepts, and why in a design system they mirror the Figma variant names
- [Figma: node IDs and links](https://help.figma.com/hc/en-us/articles/360040531773-Share-files-and-prototypes): how Copy link works and what the URL carries
- [Figma Code Connect](https://developers.figma.com/docs/code-connect/): the tooling version of this mapping
