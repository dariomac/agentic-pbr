# REQ-14 — consolidated PBR findings

Three perspectives run in parallel, blind to one another
(`tester.md`, `designer.md`, `user.md` in this directory).

**Every Decision cell below is empty on purpose.** The orchestrator is an agent
too, and the boundary in `README.md` binds it: agents find ambiguity, humans
resolve it. Filling these in is the work — and it is yours.

## Metrics

| | |
|---|---|
| Spec size | 3 sentences, 37 words |
| Blocked rows — tester | 26 |
| Blocked rows — designer | 26 |
| Blocked rows — user | 16 |
| Total blocked rows | 68 |
| Distinct spec clauses implicated | 12 (of 12 — every clause, plus the silences) |
| Ambiguity density | ~1.8 open questions per word of requirement |
| Rows corroborated by 2+ perspectives independently | 9 clusters |
| Rows found by exactly one perspective | 13 |
| Coding defects among all 68 | 0 |

## How to read the corroboration column

**3/3** — all three roles hit the same clause without seeing each other. That is
the strongest signal the technique produces: it is not one reader's hobbyhorse.
**1/3** — only one role could see it. That is what the parallel run bought you;
a single reviewer would have shipped it.

---

## C1 — "a 10% discount": the basis  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T1 · D8 · U4 | "receives a 10% discount" | 10% of what — item subtotal, subtotal plus shipping, tax-inclusive total, or the total after other discounts? | Every expected value in every test; the pricing calculation; the headline sentence of the help article | |
| D8 | "a 10% discount" | Is the discount itself taxable — is tax computed before or after it? | Tax treatment, refund arithmetic | |

## C2 — "a 10% discount": bounds and arithmetic  · **2/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T3 · D9 · U4 | "a 10% discount" | Is there a maximum discount amount? 10% of a very large order is unbounded liability. | Risk cases; finance exposure control | |
| T13 · D10 | "a 10% discount" | Is there a minimum order value, and does an order below it *consume* the bonus or leave it? | Eligibility predicate; consumption rule | |
| T2 | "a 10% discount" | Rounding rule and currency precision when 10% is not a whole cent (19.99 → 1.999)? Which way do halves go? | Boundary cases | |

## C3 — "a user refers a friend": the mechanism and attribution  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T14 · D1 · U1 | "When a user refers a friend" | By what act is a referral created — code, link, invite email, contact import? Nothing in the spec creates a referral at all. | The whole Referral entity and its state machine; steps 1–2 of the main flow; the "what do I have to do" section | |
| T14 · D2 | "a user refers a friend" | What is the attribution window between the referral and the friend's signup or first purchase? If two referrers touch the same friend, who wins — first touch or last? | Attribution rule; uniqueness constraint | |
| T12 · D2 · U2 | "a friend" | Must the referee be a brand-new account, or does an existing/dormant customer qualify? | Eligibility check at grant time | |
| T15 · D3 · U15 | "refers a friend" | Are self-referrals, shared device, shared address or shared payment instrument excluded? The spec states no anti-abuse rule; nothing forbids farming throwaway accounts. | Whether a fraud check exists in the grant path at all; the terms section | |
| D13 | "When a user refers a friend" | Is there a cap on bonuses per referrer — per month, per programme, ever? | Grant-time limits; programme cost model | |

## C4 — "completes their first purchase": the trigger  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T11 · D4 · U3 | "completes their first purchase" | Which lifecycle moment is "completes" — order placed, payment authorised, captured, shipped, delivered, or return window elapsed? These are days apart. | The trigger itself; which event the service subscribes to; the start of the 30-day clock | |
| T12 · D5 · U3 | "their first purchase" | First order ever, first *paid* order, or first non-refunded order? Does a zero-value, gift-card-funded or fully-discounted order qualify? | Eligibility predicate; stored flag vs derived query | |
| T13 · U3 | "their first purchase" | Is there a minimum qualifying order value for the friend's purchase? | Whether a £1 order triggers a bonus | |
| D6 | "their first purchase" | If the friend's first purchase happened *before* the referral existed, does it qualify retroactively? | Whether the grant is triggered by purchase or by referral; backfill | |

## C5 — "on their next purchase": consumption  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T9 · D11 · U5 | "on their next purchase" | Literally the next order placed — so a £3 order burns it — or the next order the customer chooses to apply it to? | Sequencing cases; RESERVED→CONSUMED transition; whether the article must warn customers to save it | |
| T9 · D19 | "on their next purchase" | At which lifecycle event is the bonus consumed: cart, order submitted, payment authorised, captured, or shipped? | Redemption tests; validation point in checkout | |
| T8 · D22 | "on their next purchase" | What counts as a purchase — subscription renewal, gift-card purchase, digital goods, zero-charge order? Does "checkout" include guest checkout, the app, POS? | Which surfaces must integrate | |
| T7 · D22 · U6 | "applied automatically" | May the referrer decline, remove, or defer the discount to save it for a bigger order? | Whether the bonus is user-controllable state | |

## C6 — more than one bonus: stacking and concurrency  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T4 · D12 · U7 | "receives a 10% discount" (singular) | If a referrer earns several bonuses, do they stack (20%), compound (19%), queue one per order, or overwrite? Cap per referrer or per period? | Whether bonuses are a set or a single slot; the pricing calculation; "what if I refer more than one friend?" | |
| T10 · D14 | "applied automatically at checkout" | Two concurrent checkouts by the same referrer — is the bonus reserved at checkout start and released on abandonment, or only consumed atomically at payment? Without an answer the same 10% is spendable twice. | Locking / compare-and-set strategy; the RESERVED state | |

## C7 — "applied automatically at checkout": precedence and tender  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T5 · D23 · U6 | "applied automatically at checkout" | How does it interact with coupon codes, sale prices, loyalty and other automatic promotions — combinable, mutually exclusive, best-wins? If combinable, in what order? | Position in the pricing pipeline; the amount actually charged; the most-asked support question | |
| T6 | "applied automatically at checkout" | How does it interact with store credit and gift-card tender — and is the bonus consumed when the order is fully covered by credit? | Tender-interaction cases | |

## C8 — "expire after 30 days": the clock  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T16 · D16 · U8 | "Referral bonuses expire after 30 days" | 30 days from **which** event — the referral being sent, the friend's signup, the friend's purchase completing, or bonus issuance? These can be weeks apart. | All five expiry cases; `expires_at`; "when does it stop being available?" | |
| T17 · D17 | "expire after 30 days" | 30 × 24 hours from an instant, or 30 calendar days to an end-of-day boundary? Whose timezone — referrer's, merchant's, UTC? How is a DST shift handled? | Boundary arithmetic | |
| T18 · D17 | "expire after 30 days" | Is the boundary inclusive — is a checkout at exactly S + 30 days accepted? | Boundary cases | |
| T19 · D19 | "expire after 30 days" | If the bonus is valid when the discount is applied but expired at payment capture, which timestamp governs? | The expiry race in the checkout flow | |
| D18 | "expire after 30 days" | Is expiry a stored transition written by a scheduled job, or a predicate evaluated at read time? | Whether an expiry job exists; what the account page shows between the expiry instant and the job run | |
| U16 | "expire after 30 days" | Is there any recovery for a bonus that expired unused — extension, reinstatement, support override? Sharpened by the fact that the customer was never told it existed. | "My bonus expired, can I get it back?" — a daily support question | |

## C9 — reversal and clawback  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T20 · D20 · U13 | "the friend completes their first purchase" | If the friend's qualifying purchase is refunded, cancelled or charged back while the bonus is **unused**, is the bonus revoked? | The REVOKED state; whether reversal events are consumed at all | |
| T21 · D20 · U13 | "the friend completes their first purchase" | If that purchase is reversed **after** the referrer already redeemed the bonus — clawback, write-off, or negative balance? How is the referrer charged? | CONSUMED→REVOKED; the cheapest abuse route | |
| T22 · D21 · U14 | "on their next purchase" | If the referrer's discounted order is cancelled or refunded, is the bonus reinstated — with its original expiry or a fresh 30 days? Is the refund the discounted price or list price? | Refund arithmetic; expiry recalculation | |
| T23 | "a 10% discount" | On a *partial* return of a discounted order, is the discount apportioned across returned lines, or is the line refunded at list price? | Partial-return cases | |

## C10 — visibility, notification, and the second party  · **3/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T25 · D25 · U10 | (absent from spec) | Is the referrer notified that they earned a bonus? The spec emits no events, so nothing downstream — email, account page, analytics, finance — can react. | "How will I know I have it?"; every emitted event; the customer's only discovery moment is currently spending it | |
| T25 · D22 · U9 | (absent from spec) | Where can a customer see a pending bonus, its value, and its expiry *before* checkout? No screen, list or account section is specified. | Any instruction to "use it before it expires" — the customer is asked to watch an invisible clock | |
| U11 | "The discount is applied automatically at checkout" | Is the discount **shown and labelled** at checkout, or merely applied to the price? "Applied" does not imply visible. | Whether a customer can tell a referral bonus from a sale price | |
| T26 · D2 · U12 | "the referring user receives" | Does the referred friend receive anything, or see that they were referred? The second party appears only as a trigger. | The friend-facing half of the article; any invite copy. A test asserting the friend gets nothing rests on reading silence as intent | |

## C11 — the referrer's own standing  · **2/3**

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| T24 · D24 | "the referring user receives" | Must the referrer be active and in good standing at redemption? What if the account is deactivated, banned, merged or deleted between earning and redeeming? | Eligibility at grant and at redemption; data retention | |

## C12 — delivery, idempotency, failure  · **1/3**

Only the designer could see this cluster. It is invisible from a test table and
invisible from a help article — and it is where the money leaks.

| # | Spec text | Question | Blocks | Decision |
|---|-----------|----------|--------|----------|
| D7 | "the referring user receives a 10% discount" | Is the bonus a stored, addressable object with its own lifecycle, or a value derived at checkout from referral history? | Whether a ReferralBonus entity exists — and with it the grant transaction, expiry job, idempotency key and clawback path, or none of them | |
| D15 | "completes their first purchase" | If the "first purchase completed" event is delivered twice, what prevents a second bonus? What is the dedupe key — referral, referee, or order? | Idempotency of the grant handler under at-least-once delivery and replay | |
| D26 | (absent from spec) | What happens on partial failure — bonus write fails after the friend's order commits, payment fails after the discount is applied, the upstream event never arrives? Outbox, retry, reconciliation sweep? | Delivery guarantees; whether anything ever detects a bonus that should exist and does not | |

---

## What the parallel run bought

Thirteen rows were found by exactly one perspective. They are the argument for
running three readers instead of one good one:

- **Only the tester** asked about rounding and currency precision (T2),
  gift-card and store-credit tender (T6), and apportioning a discount across a
  partial return (T23). These come from having to write a literal expected
  value.
- **Only the designer** asked whether the bonus is stored or derived (D7),
  what makes the grant idempotent under duplicate delivery (D15), whether
  expiry is a job or a read-time predicate (D18), how a double-spend is
  prevented across concurrent checkouts (D14), whether a pre-referral purchase
  qualifies retroactively (D6), and what reconciles a partial failure (D26).
- **Only the user** noticed that "applied" does not promise "visible" (U11),
  that a customer is being asked to watch a clock they cannot see (U9), and
  that support will be asked daily to reinstate bonuses the customer never knew
  they had (U16).

None of the three would have produced the other two's list. That is the claim
PBR makes, and this run is consistent with it.

## Check against the reference

`findings/REQ-14-expected-findings.md` lists 21 rows. This run reproduced **20
of 21 as distinct rows**, and the 21st — "is the referrer warned *before*
expiry?" — is present only implicitly, folded into U9's "no surface shows the
expiry at all". Worth adding to the reference as a separate row, and worth
tracking as the one a live run can miss.

It also found roughly two dozen questions the reference does not carry: tax
treatment, attribution windows and double attribution, retroactive
qualification, minimum order value, gift-card tender, partial-return
apportionment, reservation and double-spend, the apply-vs-capture expiry race,
per-referrer caps, referrer account deletion, and post-expiry support recovery.

## Next step — yours, not the agent's

Fill in the Decision column. Then, and only then, write the Gherkin.

Nothing above is a coding defect. A coding agent handed REQ-14 would emit clean,
readable, internally consistent scenarios — with a silent decision on all 68
rows. Green suite, wrong product.
