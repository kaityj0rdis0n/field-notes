---
title: Presence is not validity
category: software
date: 2026-08-23
tags: [rails, validation, activerecord, mongoid]
---

# Presence is not validity

## TL;DR

"The field has a value" and "the value is acceptable" are two different validations. Frameworks only give you the first one for free, and zero passes it.

## The explanation

In Rails, presence means "not blank," and blank is a short, specific list: `nil`, `false`, empty strings, and empty collections. Zero is not on that list. `0.present?` is true, and so is `0.0.present?`.

So a model like this accepts a zero donation:

```ruby
class Donation
  validates_presence_of :amount
end
```

It reads as: "amount must be filled in." Walk two values through it. `amount = nil`: blank, validation fails, correct. `amount = 0`: present, validation passes, and a zero-dollar donation saves cleanly. The developer who wrote the validation almost certainly meant "a donation must have a real amount," but presence only checks that *something* is there, not that the something makes sense.

The validation that says what was actually meant:

```ruby
class Donation
  validates_presence_of :amount
  validates :amount, numericality: { greater_than: 0 }
end
```

Presence answers "is it filled in?" Numericality answers "is it a number in the acceptable range?" They are separate questions with separate validators, and a numeric field usually needs both.

## Habits and when this bites

Whenever a field is numeric, ask what the acceptable range is, not just whether it's required. Amounts, percentages, quantities, and durations almost always have a real lower bound that presence will not enforce. The bug this produces is quiet: nothing crashes, the record saves, and the nonsense value only shows up downstream, in a report that doesn't add up or a customer-facing screen that displays something absurd.

One caution before adding a range validation to an existing model: it runs on *every* subsequent save, including saves triggered by unrelated updates. If bad values already exist in the table, the new validation makes those records unsaveable until the data is cleaned up. Audit first, then validate.

## See also

- [Truthiness and form inputs](./truthiness-form-inputs.md)
- [Validation has to live at every door](./validation-at-every-door.md)
