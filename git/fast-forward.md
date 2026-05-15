---
title: Fast-forward (git)
category: git
date: 2026-05-15
tags: [git, version-control, merge]
---

# Fast-forward (git)

## TL;DR

A "fast-forward" is when your local branch is behind the remote by some commits, and those commits sit directly on top of where you are — no diverging history. Git just slides your branch pointer forward along the line. No merge commit, no possibility of conflict.

## The explanation

Two scenarios:

**Fast-forward (clean line)**
```
Before:  A → B → C        (your local main)
                  ↑
                  here
Remote:  A → B → C → D → E    (origin/main)

After:   A → B → C → D → E    (your local main now matches)
                          ↑
                          here
```
Git slides the `main` pointer from C to E. No new commit, no merge.

**Not a fast-forward (diverging history)**
```
Yours:   A → B → C → X        (you added X locally)
Remote:  A → B → C → D → E    (origin added D and E)
```
You and the remote both moved forward from C, in different directions. Git can't just slide — you'd need a merge commit (or a rebase) to combine X with D and E.

## When does this matter?

- **After someone merges a PR while you're on `main`**: when you `git pull`, your local main usually fast-forwards. Clean, no merge commit.
- **When you've made local commits and someone else has pushed**: you can't fast-forward. You'll need to rebase (`git pull --rebase`) or merge.
- **Forcing a merge commit anyway**: `git merge --no-ff` — useful when you want a visible record of the merge in history, even when fast-forward is possible.
- **Reading commit history**: a long string of commits with no merge commits often means everything has been fast-forwarded — that's a healthy, linear history.

It's the cleanest possible "your local is now up to date" outcome.

## See also

- `git merge --no-ff` flag
- [Git docs — Basic merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
