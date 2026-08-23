---
title: Validation has to live at every door
category: software
date: 2026-08-23
tags: [validation, architecture, api-design, defense-in-depth]
---

# Validation has to live at every door

## TL;DR

A check that lives only in the form protects only the form. Every other write path walks straight past it.

## The explanation

A web form is rarely the only way data gets into a table. The same record type is usually written by several doors: the main UI form, a bulk CSV import, a public or internal API, an admin tool, a mobile app, a migration script. Each door is a separate code path, and a validation added to one door does nothing for the others.

The failure mode is easy to picture with a newsletter signup. The form validates email format in the browser, so the team considers email format "handled." Then someone bulk-imports a spreadsheet of addresses through a different endpoint, nothing validates them, and the table fills with typos that bounce forever. The form was never broken. It was just one door of three.

This also fails in the other direction: a validation can exist and be *broken* at one door while working at another. Two forms editing the same kind of record can drift, one rejecting a bad value and one accepting it, and the difference goes unnoticed because each form looks fine in isolation. The only view that catches either failure is the list of doors, not any single door.

The question that finds these holes during review is:

> What are all the paths that write this field?

Not "does the form validate it?" but "what else writes it?" Answering honestly usually turns up at least one path nobody thought about, and it's often the bulk import.

## Where the authoritative check belongs

Put the check that actually matters as close to the data as you can afford: the API layer, the service that owns the write, or the database constraint. Checks in the UI are still worth having, but their job is user experience, giving fast feedback before a round trip. They are courtesy copies of the rule, not the rule.

A useful way to hold it: the frontend validates for *kindness*, the backend validates for *truth*. If the two ever disagree, the backend's answer is the one the data lives with.

## Habits and when this bites

When you add or fix a validation, list the write paths first and fix all of them in the same change, or say explicitly which doors remain open and why. When you review someone else's validation change, ask the door question before approving. And treat "the UI prevents that" in a design discussion as an unfinished sentence: the UI prevents that *through the UI*.

## See also

- [Truthiness and form inputs](./truthiness-form-inputs.md)
- [Presence is not validity](./presence-is-not-validity.md)
- [Contract mismatch: fix the caller](./contract-mismatch-fix-the-caller.md)
