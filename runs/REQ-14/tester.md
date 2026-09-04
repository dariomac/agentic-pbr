# PBR — Tester — REQ-14

Spec under inspection: `specs/REQ-14.md` (Referral bonus, three sentences).
Currency assumed to be whatever the store already uses; I write `USD` only as a
placeholder label on cart amounts so cases are concrete. `?` marks every place I
could not write a checkable expected value from the spec text.

## Artifact: acceptance test cases

### A. Earning the bonus

**TC-01 — Bonus is issued when the referred friend completes a first purchase**
- Given user R referred friend F, and F has no prior purchases
- When F places an order for USD 50.00 and it reaches the state the spec calls
  "completes" (= ?)
- Then R holds exactly 1 referral bonus, with
  `status = ?` (no state vocabulary in the spec),
  `issued_at = ?` (which of order-placed / payment-captured / shipped /
  delivered / return-window-closed is the trigger),
  `expires_at = ?`
- And the bonus is observable to R via ? (spec defines no balance, no
  notification, no surface where the bonus is visible before checkout)

**TC-02 — No bonus before the friend's first purchase**
- Given R referred F, and F has signed up but placed no order
- When R checks out a cart with subtotal USD 100.00
- Then order total = USD 100.00 + tax + shipping, discount applied = USD 0.00
- (This case is complete: the spec makes the friend's purchase a precondition.)

**TC-03 — Friend is an existing customer**
- Given F already has 3 prior orders on the platform, then accepts R's referral
- When F places another order for USD 50.00
- Then number of bonuses held by R = ? ("their first purchase" — first ever, or
  first after accepting the referral?)

**TC-04 — Referral attribution window**
- Given R sends a referral to F on 2026-01-01; F signs up on 2026-04-01 and buys
  on 2026-04-02
- Then number of bonuses held by R = ? (no attribution window in the spec, and
  no definition of what act constitutes "refers")

**TC-05 — Self-referral / same person, second account**
- Given R creates a second account F2 with a different email but the same
  payment instrument and device, and refers it
- When F2 completes a first purchase of USD 50.00
- Then number of bonuses held by R = ? (no eligibility or abuse rule stated)

**TC-06 — Friend's qualifying order is below any threshold**
- Given F's first purchase is USD 0.99
- Then number of bonuses held by R = ? (no minimum qualifying spend stated —
  1 is defensible, 0 is defensible)

### B. Redeeming the bonus — amount

**TC-07 — Happy path, automatic application at checkout**
- Given R holds 1 unexpired bonus
- And cart = 1 item at USD 100.00, tax 8.25% , shipping USD 9.99
- When R reaches checkout
- Then discount line = ? and order total = ?
  - candidate A (10% of item subtotal): discount 10.00, total 107.42
  - candidate B (10% of subtotal + shipping): discount 11.00, total 116.05
  - candidate C (10% of the tax-inclusive grand total 118.24): discount 11.82,
    total 106.42
  - the spec does not say what the 10% is a percentage **of**
- And the discount appears without R entering a code — this half is checkable.

**TC-08 — Rounding**
- Given R holds 1 unexpired bonus and cart subtotal = USD 19.99
- When R checks out
- Then discount = ? (1.999 → 2.00 / 1.99 / 1.999 carried into the total; no
  rounding rule, no precision, no half-way tie-break in the spec)

**TC-09 — Large order / discount cap**
- Given R holds 1 unexpired bonus and cart subtotal = USD 5,000.00
- Then discount = ? (500.00 if uncapped; the spec states no cap, but states no
  "uncapped" either — a max-discount rule is exactly the kind of thing that is
  normally implicit)

**TC-10 — Very small order**
- Given R holds 1 unexpired bonus and cart subtotal = USD 0.30
- Then discount = ? and bonus is consumed = ? (no minimum order value; a
  0.03 discount that burns the whole bonus is defensible, so is refusing to
  apply it)

**TC-11 — Non-discountable contents**
- Given cart = one gift card at USD 100.00 (or a subscription renewal, or a
  digital item)
- Then discount = ? ("purchase" is undefined — whether a renewal or a gift-card
  purchase is the "next purchase" that consumes the bonus)

### C. Redeeming the bonus — stacking and precedence

**TC-12 — Two referrals, one checkout**
- Given F1 and F2 have each completed a first purchase, so R has earned two
  bonuses; cart subtotal = USD 100.00
- When R checks out
- Then discount = ? (20.00 additive / 19.00 sequential / 10.00 one-per-order
  with the second retained / 10.00 with the second forfeited) and remaining
  bonuses = ?

**TC-13 — Bonus plus an existing promo code**
- Given R holds 1 unexpired bonus, cart subtotal USD 100.00, and R enters coupon
  `SAVE20` (20% off)
- When R checks out
- Then discount = ? and total = ? (combinable? mutually exclusive, best-wins?
  applied in which order — 100 → 80 → 72, or 100 → 90 → 72, or 100 → 70; and is
  the bonus consumed if the coupon wins?)

**TC-14 — Bonus plus store credit / gift-card tender**
- Given R holds 1 unexpired bonus, cart subtotal USD 100.00, and USD 100.00 of
  store credit on the account
- Then amount charged to card = ?, credit consumed = ?, bonus consumed = ?

**TC-15 — R does not want the discount on this order**
- Given R holds 1 unexpired bonus and is buying one USD 3.00 item, intending to
  save the bonus for a larger order next week
- When R checks out
- Then discount applied = ? and bonus remaining = ? ("applied automatically"
  suggests R has no say; the spec never says the application can be declined,
  nor that it cannot)

### D. State transitions

**TC-16 — Bonus is consumed by the next purchase**
- Given R holds 1 unexpired bonus and completes discounted order A
- When R places order B for USD 100.00 the next day
- Then discount on order B = USD 0.00 (supported: "a 10% discount on their next
  purchase", singular)
- And the point at which the bonus became unavailable = ? (cart, order placed,
  payment authorized, payment captured, shipped)

**TC-17 — Two checkouts at the same instant**
- Given R holds 1 unexpired bonus and submits order A and order B from two
  devices within the same second
- Then number of discounted orders = 1 (supported by "next purchase") and the
  order that receives it = ? (no tie-break rule; also depends on TC-16's
  consumption point)

**TC-18 — Checkout spans the expiry boundary**
- Given R's bonus expires at instant X (X itself = ?), R opens checkout at
  X − 2 min and payment is captured at X + 3 min
- Then discount applied = ? (which timestamp is tested against expiry: cart
  creation, checkout entry, order submission, payment capture)

### E. Expiry

**TC-19 — Within the window**
- Given R's bonus was created by F's qualifying purchase on 2026-03-01 at
  10:00 (timezone = ?)
- When R checks out on 2026-03-20 with subtotal USD 100.00
- Then discount = ? (per TC-07; the *eligibility* half is clear here, the amount
  is not)

**TC-20 — Exact boundary, day 30**
- Given the bonus clock starts at instant S (S = ? — referral sent / friend's
  signup / friend's purchase completing / bonus issuance)
- When R checks out at exactly S + 30 days
- Then discount applied = ? (inclusive or exclusive boundary is not stated)

**TC-21 — Day 31**
- Given the same bonus
- When R checks out at S + 30 days + 1 minute
- Then discount = USD 0.00 (expired) — checkable *only once S is defined*;
  S = ?

**TC-22 — Calendar days vs 24-hour periods, and timezone**
- Given the clock starts 2026-03-01 at 23:30 in R's local time (UTC+13), i.e.
  2026-03-01 10:30 UTC, and R's region observes a DST shift on 2026-03-29
- Then `expires_at` = ? (30 × 24 h from the instant / end of the 30th calendar
  day / in R's timezone / merchant timezone / UTC)

**TC-23 — Expired and unused**
- Given R never purchases within the window
- Then bonus status at S + 31 days = ? (no state vocabulary), R is notified = ?
  (spec says nothing about notification, so there is no observable to assert on
  other than a later checkout total)

### F. Preconditions undone

**TC-24 — Friend's qualifying purchase is refunded, bonus not yet used**
- Given F's first purchase is fully refunded 5 days later; R has not checked out
- When R checks out with subtotal USD 100.00 on day 6
- Then discount = ? (bonus revoked, or kept because it was "completed" once)

**TC-25 — Friend's qualifying purchase is refunded after R redeemed the bonus**
- Given R already used the bonus on order A (paid USD 90.00 + tax), then F's
  purchase is refunded / charged back
- Then order A's total = ?, amount additionally charged to R = ?, R's bonus
  balance = ? (no clawback rule either way)

**TC-26 — Friend cancels before fulfilment**
- Given F places a first order and cancels it before it ships
- Then number of bonuses held by R = ? (depends on the "completes" event in
  TC-01)

**TC-27 — R's discounted order is cancelled or fully refunded**
- Given R used the bonus on order A and returns everything for a full refund on
  day 12
- Then refund amount = ? (90.00 paid back, or 100.00 list price), bonus
  reinstated = ?, and if reinstated, `expires_at` = ? (original expiry, or a new
  30 days)

**TC-28 — Partial refund of a discounted order**
- Given order A = two items at USD 50.00 each, 10% applied, R paid USD 90.00
  (+ tax), and R returns one item
- Then refund amount = ? (45.00 apportioned, or 50.00 at line price) and
  remaining discount on the retained item = ?

**TC-29 — R's account is deactivated between earning and redeeming**
- Given R's account is deactivated on day 3 and reactivated on day 10
- When R checks out on day 11
- Then discount = ? (no eligibility-at-redemption rule)

### G. Direction / anti-inversion

**TC-30 — The discount goes to the referrer, not the referred friend**
- Given R referred F, F completed a first purchase, and F now places a second
  order of USD 100.00 while R also places an order of USD 100.00
- Then F's discount = USD 0.00 and R's discount = > 0
- (Checkable, and it fails if the implementation rewards the wrong party. The
  exact value of R's discount is blocked by TC-07, but the *direction* assertion
  stands on its own. Caveat: F's expected 0.00 relies on reading the spec's
  silence about the friend as "the friend gets nothing".)

## Perspective questions

**1. Which cases could you not finish, and what exact value was missing?**

All of TC-01, TC-03 to TC-15, TC-17 to TC-20, TC-22 to TC-29 are blocked in
whole or in part. The missing values, deduplicated:

- the base the 10% is taken from (item subtotal / + shipping / tax-inclusive
  total) — without it no single test in section B has a `Then`
- the rounding rule and precision for a non-whole-cent 10%
- whether there is a discount cap or a minimum qualifying order
- the instant the 30-day clock starts, its timezone, its unit (24 h periods vs
  calendar days), and whether the boundary is inclusive
- the lifecycle event that counts as the friend "completing" a first purchase
- the lifecycle event that consumes the bonus on the referrer's side
- the stacking rule and precedence order against coupons and store credit
- whether multiple earned bonuses accumulate, and any cap
- the reversal rules in every direction (friend refund, referrer refund,
  partial refund)
- a state vocabulary for the bonus, so a test can assert anything other than a
  checkout total

**2. For every number in the spec — is its unit, base, and rounding defined?**

There are exactly two numbers.

- "10%" — unit is clear (a percentage). **Base is undefined** (TC-07).
  **Rounding is undefined** (TC-08). No cap, no floor, no currency handling
  (TC-09, TC-10).
- "30 days" — unit is nominally clear but ambiguous in practice: 30 × 24 h or 30
  calendar days (TC-22). **Base (the start event) is undefined** (TC-20).
  Rounding to a day boundary, if any, is undefined, as is the timezone.

So: zero of two numbers are fully specified.

**3. For every time period — is the start event defined? The timezone?
Inclusive or exclusive of the boundary?**

One time period: "expire after 30 days".
- Start event: **not defined.** Four defensible candidates (referral sent,
  friend's signup, friend's qualifying purchase, bonus issuance), and they can
  be weeks apart.
- Timezone: **not defined.** Referrer local, merchant, or UTC — a bonus issued
  late in the evening changes its last usable day depending on the answer. DST
  is not addressed.
- Boundary: **not defined.** Whether a checkout at exactly S + 30 days is
  accepted is a coin flip (TC-20), as is which timestamp in the checkout flow is
  compared against it (TC-18).

**4. Which cases have more than one defensible expected result?**

TC-01, TC-03, TC-04, TC-05, TC-06, TC-07, TC-08, TC-09, TC-10, TC-11, TC-12,
TC-13, TC-14, TC-15, TC-17, TC-18, TC-19, TC-20, TC-22, TC-23, TC-24, TC-25,
TC-26, TC-27, TC-28, TC-29. The sharpest ones — where two implementers would
each ship confidently and disagree — are TC-07 (base), TC-12 (stacking),
TC-13 (precedence), TC-20 (boundary) and TC-24/TC-25 (revocation).

**5. What happens when something the spec treats as final is reversed?**

The spec treats two events as final and defines the reversal of neither:

- The friend's first purchase is treated as a permanent trigger. Refund,
  cancellation and chargeback are unaddressed — both before redemption (TC-24,
  TC-26) and after (TC-25, where a clawback would mean charging a customer more
  than they agreed to at checkout).
- The referrer's redemption is treated as permanent. A cancelled or refunded
  discounted order leaves it undefined whether the bonus returns, what refund
  amount is owed, and how a partial return apportions a percentage discount
  (TC-27, TC-28).

Retry is also unaddressed: a failed payment that is retried successfully, or a
checkout abandoned after the discount was displayed, may or may not have
consumed the bonus (TC-16, TC-18).

**6. Could you write a test that would fail if the feature were implemented
backwards?**

Partly, and only on one axis.

- **Direction — yes.** TC-30 fails if the discount is given to the referred
  friend instead of the referrer, and TC-02 fails if the bonus is granted on
  referral rather than on the friend's first purchase. Those two assertions are
  real.
- **Amount — no.** I cannot write a test that fails for a wrong-but-plausible
  implementation, because I cannot state the right one. An implementation
  charging 107.42, 116.05 or 106.42 on the TC-07 cart all satisfy the sentence
  as written.
- **Expiry — no.** With no start event and no timezone, an implementation that
  expires the bonus on day 30 from the referral click and one that expires it on
  day 30 from the friend's purchase both satisfy the sentence, and they differ
  by however long the friend took to buy.

So the requirement is testable for *who* and *whether*, and not testable for
*how much* or *until when* — which is most of its behaviour.

## Blocked rows

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|
| T1 | "receives a 10% discount" | 10% of what? Item subtotal, subtotal + shipping, or the tax-inclusive total? | TC-07, TC-19, and the expected value of every case in section B | |
| T2 | "a 10% discount" | What rounding rule and precision apply when 10% is not a whole cent (19.99 → 1.999)? Which way do halves go? | TC-08, TC-10 | |
| T3 | "a 10% discount" | Is there a maximum discount amount? A minimum order value below which it is not applied? | TC-09, TC-10 | |
| T4 | "a 10% discount on their next purchase" | If a referrer earns two or more bonuses, do they stack on one order (20%? 19%?), queue one per order, or is only one ever held? Is there a cap per referrer or per period? | TC-12 | |
| T5 | "The discount is applied automatically at checkout" | How does it interact with coupon codes and other promotions — combinable, mutually exclusive, best-wins? If combinable, in what order are they applied? | TC-13 | |
| T6 | "The discount is applied automatically at checkout" | How does it interact with store credit / gift-card tender, and is the bonus consumed when the order is fully covered by credit? | TC-14 | |
| T7 | "applied automatically" | May the referrer decline or defer the discount to save it for a larger order? | TC-15 | |
| T8 | "on their next purchase" | What counts as a purchase — subscription renewal, gift-card purchase, digital goods, a zero-charge order? | TC-11 | |
| T9 | "on their next purchase" | At which lifecycle event is the bonus consumed: cart, order submitted, payment authorized, payment captured, or shipped? | TC-16, TC-17, TC-18 | |
| T10 | "on their next purchase" | If two checkouts are submitted concurrently, which one gets the discount? | TC-17 | |
| T11 | "the friend completes their first purchase" | What event is "completes": order placed, payment captured, shipped, delivered, or return window elapsed? | TC-01, TC-26 | |
| T12 | "their first purchase" | First purchase ever on the platform, or first after accepting the referral? Is an existing customer eligible as a referred friend? | TC-03 | |
| T13 | "their first purchase" | Is there a minimum qualifying order value for the friend's purchase? | TC-06 | |
| T14 | "a user refers a friend" | What act constitutes a referral (link click, code at signup, email invite), and what is the attribution window between it and the friend's signup or first purchase? | TC-01, TC-04 | |
| T15 | "a user refers a friend" | What eligibility / anti-abuse rules apply — self-referral, shared payment instrument or device, referring an existing account? | TC-05 | |
| T16 | "Referral bonuses expire after 30 days" | 30 days from **which** event: the referral being sent, the friend's signup, the friend's first purchase completing, or bonus issuance? | TC-19, TC-20, TC-21, TC-22, TC-23 | |
| T17 | "expire after 30 days" | 30 × 24-hour periods from an instant, or 30 calendar days to an end-of-day boundary? In which timezone — referrer's local, merchant's, or UTC? How is a DST shift handled? | TC-22 | |
| T18 | "expire after 30 days" | Is the boundary inclusive or exclusive — is a checkout at exactly S + 30 days accepted? | TC-20 | |
| T19 | "expire after 30 days" | Which timestamp in the checkout flow is compared against expiry (cart creation, checkout entry, order submission, payment capture) when the flow spans the boundary? | TC-18 | |
| T20 | "the friend completes their first purchase" | If the friend's qualifying purchase is refunded, cancelled or charged back and the bonus is **unused**, is the bonus revoked? | TC-24, TC-26 | |
| T21 | "the friend completes their first purchase" | If that purchase is reversed **after** the referrer already redeemed the bonus, is the discount clawed back, and how is the referrer charged? | TC-25 | |
| T22 | "on their next purchase" | If the referrer's discounted order is cancelled or fully refunded, is the bonus reinstated, and with the original expiry or a fresh 30 days? What refund amount is owed — the discounted price or list price? | TC-27 | |
| T23 | "a 10% discount" | On a partial return of a discounted order, is the discount apportioned across returned lines or is the line refunded at list price? | TC-28 | |
| T24 | "the referring user receives" | Must the referrer's account be active/in good standing at redemption? What happens if it is deactivated, banned or deleted between earning and redeeming? | TC-29 | |
| T25 | whole spec | The spec names no observable state for a bonus (issued / active / used / expired), no balance surface, and no notification, so nothing can be asserted before checkout and expiry is unobservable until the referrer happens to buy. Is any of that in scope? | TC-01, TC-23 | |
| T26 | "the referring user receives a 10% discount" | Does the referred friend receive anything? A test asserting the friend gets 0 rests on reading the silence as "nothing". | TC-30 | |
