---
title: Merchant of Record vs. Principal/Agent
category: accounting-tax
date: 2026-05-15
tags: [accounting, payments, revenue-recognition, marketplace]
---

# Merchant of Record vs. Principal/Agent

## TL;DR

Two questions people conflate but shouldn't:

- **Merchant of Record (MoR)** is a *legal* designation — who is on the hook to the buyer for the transaction.
- **Principal vs. Agent** is an *accounting* designation — whether you report gross revenue or just your slice.

You can be MoR without being the principal. Marketplace platforms run into this constantly.

## The explanation

### Merchant of Record (MoR)

The MoR is the legal counterparty to the buyer. The MoR:

- Collects the money on the credit card.
- Owns the chargeback liability.
- Is the legal seller for sales-tax purposes (and is often required to collect / remit it).
- Appears on the buyer's credit card statement.

For a ticketing platform, being MoR means: when a buyer pays $100 for a concert ticket, the $100 hits *your* bank account first, even though most of it will be paid out to the venue.

### Principal vs. Agent

This is a *revenue recognition* question, governed by [ASC 606-10-55-36 through 55-40](https://asc.fasb.org/) under GAAP. The test is: who controls the specified good or service before it's transferred to the customer?

- **Principal** = you control the service → report **gross** revenue (the whole $100), then recognize the venue payout as cost of goods sold.
- **Agent** = you facilitate someone else's service → report **net** revenue (just your service fee, say $10), with the $90 face value flowing through as a pass-through liability.

Indicators of being the principal (paraphrased from ASC 606):

- You have primary responsibility for delivering the service.
- You bear inventory risk before transfer.
- You have discretion in setting the price.

### Why they're separate

You can be MoR but accounting-wise an agent: you collect the full $100, but you only report $10 as revenue because you don't actually control the concert. The $90 sits on your balance sheet as a payable to the venue.

You can also be MoR and the principal: you fully control the experience, set the price, bear the risk. Report the full $100 as revenue and the $90 venue payout as COGS.

For tax, the **AFS (Applicable Financial Statement) conformity rule** at [IRC § 451(b)](https://www.law.cornell.edu/uscode/text/26/451) generally requires tax treatment to follow the book treatment.

## When does this matter?

- **Marketplaces (tickets, real estate, ridesharing, e-commerce platforms)**: this is the central characterization question. It changes reported revenue by an order of magnitude.
- **Investor optics**: gross revenue companies look much bigger than net revenue companies. People have been sloppy / aggressive about this historically.
- **Sales tax / 1099-K obligations**: being MoR triggers these regardless of principal/agent — you're the one collecting money.
- **Refunds / chargebacks**: the MoR is on the hook even if it's an agent on revenue.

## The pattern people get wrong

The classic mess: treating the full ticket price as revenue when collected, instead of separating (a) the platform's service fee from (b) face value held in trust. That inflates revenue, distorts the effective tax rate, and creates large reversals on refunds.

MoR status and principal/agent status are separate questions. Being MoR doesn't automatically make you the principal.

## See also

- [Revenue recognition (accrual basis)](revenue-recognition.md)
- [1099-K reporting](../payments/1099-k.md) — flows from being MoR
- [ASC 606-10-55-36 through 55-40](https://asc.fasb.org/) — principal vs. agent indicators
