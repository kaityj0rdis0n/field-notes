---
title: Semantic versioning (semver)
category: software
date: 2026-05-20
tags: [versioning, releases, semver, software]
---

# Semantic versioning (semver)

## TL;DR

A version number like `v1.2.3` is three numbers separated by dots. **MAJOR.MINOR.PATCH**. Each number signals something different about what changed:

- **MAJOR** (`1.x.x`) — *broke* something that used to work. Read the changelog before upgrading.
- **MINOR** (`x.2.x`) — *added* new stuff. Old stuff still works.
- **PATCH** (`x.x.3`) — *fixed* a bug or typo. No new features.

The version number isn't about how "big" the release feels — it's a **promise to the future** about how much will change between versions.

## The explanation

### What each number signals

Think of a phone OS:

| Change | Example | What it means |
|---|---|---|
| MAJOR | `iOS 17 → iOS 18` | Settings menu moved around. Some apps need updates. |
| MINOR | `iOS 17.0 → iOS 17.1` | New emoji, new features. Everything still works. |
| PATCH | `iOS 17.1.1 → iOS 17.1.2` | Battery bug fix. |

You probably don't read the changelog for a patch. You skim it for a minor. You definitely read it for a major.

### The "v0.x.x" thing

Common confusion: *isn't a first release a "major" thing?*

No. "MAJOR" in semver doesn't mean "big or important." It means "breaks backward compatibility with the previous version." For a *first* release there's no previous version to break — so the major number is really a promise about the future.

- **`v1.0.0` says:** "I'm committing to this design. If I break it later, I owe you a heads-up by bumping to `v2.0.0`."
- **`v0.1.0` says:** "Still iterating. Things might move around. Don't assume stability yet."

Lots of projects ship a first release at `v0.1.0` and stay in `v0.x` while they figure things out. Some never reach `v1.0.0`. Both are normal.

### Worked example

You ship a small library at `v0.1.0`. Then:

| Change | New version |
|---|---|
| Fix a typo in an error message | `v0.1.1` |
| Add an optional new config option | `v0.2.0` |
| Rename a public function people were using | `v0.3.0` (in `v0.x`, anything goes between minors) or `v1.0.0` if you're ready to commit |
| Big rewrite — entire API changes | `v1.0.0` (first stable release) or `v2.0.0` if you'd already shipped `v1.0.0` |

## When does this matter?

- **Tagging GitHub releases** — `gh release create v0.1.0` or `git tag v0.1.0 && git push --tags`
- **Reading dependency upgrade reports** — Dependabot shows "bumped from `2.3.1` to `2.3.4`" — the patch bump tells you it's probably safe to merge
- **`package.json` version ranges in npm** — `^1.2.3` allows minor + patch updates; `~1.2.3` allows only patch
- **Deciding when to ship `v1.0.0`** — when you're confident enough in your design that breaking it would cost more than it gains. No rush.

## See also

- [semver.org](https://semver.org/) — the canonical spec
- [npm docs on version ranges](https://docs.npmjs.com/about-semantic-versioning) — how npm interprets `^`, `~`, etc.
