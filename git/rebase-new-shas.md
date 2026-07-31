---
title: Rebase and new SHAs (git)
category: git
date: 2026-06-18
tags: [git, rebase, merge, version-control]
---

# Why rebase creates new SHAs (and why branch -d fails after one)

## TL;DR

Rebase rewrites history by replaying commits on a new base — same content, but brand new commit objects with new SHAs. This is why `git branch -d` complains a branch is "not fully merged" even when you know the content is on main.

## The explanation

When you **merge**, git takes your commits as-is and joins them into the target branch. The original commit objects — and their SHAs — are preserved in history.

When you **rebase**, git replays each of your commits one by one on top of the new base, as if you'd written them there all along. Each replay creates a brand new commit object with a new SHA, even if the diff is identical.

So after `git rebase main` followed by `git merge --ff-only` and `git push`, your branch's *content* is on main — but the original commit SHAs are not. When you then run `git branch -d my-branch`, git looks for those original SHAs in main's ancestry, doesn't find them (because they were replaced by rebased equivalents), and flags the branch as "not fully merged."

The fix: `git branch -D` to force-delete. It's safe when you've already confirmed the content landed on the remote.

```bash
# rebase rewrites: original SHA abc1234 becomes xyz5678 on main
git rebase main
git checkout main
git merge --ff-only my-branch
git push origin main

# git -d fails — can't find abc1234 in history
git branch -d my-branch  # error: not fully merged

# force delete is safe here — content is on remote
git branch -D my-branch  # Deleted branch my-branch (was xyz5678)
```

## When does this matter?

- Any time you rebase a feature branch before merging and then try to clean up the local branch with `git branch -d`.
- When reviewing someone else's PR and wondering why "merge" vs "rebase and merge" produces different commit graphs.
- When using `git log` to track down a specific commit — if the branch was rebased, the SHA you copied from an earlier point won't exist in main's history even if the change is there.

## See also

- [Fast-forward merges](fast-forward.md) — the merge case this contrasts with
- `git branch -d` vs `git branch -D` — `-d` is safe delete (checks merge status), `-D` is force delete
- `git log --oneline main..my-branch` — useful for seeing which commits are on a branch vs main before rebasing/merging
