# PBR — Designer — REQ-14

Spec under inspection: `specs/REQ-14.md` (three sentences).
I am designing this before writing code. Where the spec does not give me a
field, a state, or a transition, I have written `?` and stopped — I have not
chosen a value to keep the diagram whole.

## Artifact: high-level design

### Entities and fields

**`Referral`** — the link between referrer and referee. The spec implies it
("when a user refers a friend") but never describes how one comes into
existence, so every field below is `?` except the two parties.

| Field | Type | Source in spec |
|---|---|---|
| `id` | id | — |
| `referrer_user_id` | fk User | "a user refers" |
| `referee_user_id` | fk User | "a friend" — but see `?` below |
| `referral_mechanism` | `?` | spec names no code, link, invite email, or contact import |
| `created_at` | timestamp | `?` — no creation trigger is described |
| `attributed_at` | timestamp | `?` — when does a referee become *attributed* to a referrer? At signup? At first purchase? |
| `status` | enum | `?` — see state machine |
| `expires_at` | timestamp | `?` — the "30 days" may attach here or to the bonus; see D16 |

Unresolvable at this level: whether `referee_user_id` must point at a *new*
account, whether a referee may be attributed to more than one referrer, and
what happens if the friend already had an account with purchase history. All `?`.

**`ReferralBonus`** — the 10% discount. **I cannot tell whether this entity
exists at all.** "The referring user receives a 10% discount on their next
purchase" is ambiguous between:

- (a) a stored row created when the friend's first purchase completes, with a
  lifecycle and an expiry; and
- (b) a value derived at checkout by asking "does this user have a qualifying
  referral in the last 30 days?"

These are not interchangeable designs — (a) needs a grant event, a consumption
transaction, an expiry job, and idempotency keys; (b) needs none of those but
cannot express "used" or "expired" as facts, cannot be shown in an account
page as a balance, and cannot be clawed back. I have drawn (a) below **only to
expose the fields it would need**, not because the spec chose it.

| Field | Type | Source in spec |
|---|---|---|
| `id` | id | — |
| `referrer_user_id` | fk User | "the referring user" |
| `source_referral_id` | fk Referral | implied |
| `percent` | decimal | `10` — the only literal the spec gives me |
| `discount_basis` | enum | `?` — 10% of what? subtotal / +shipping / +tax / post-other-discounts (D8) |
| `max_discount_amount` | money | `?` — no cap stated (D9) |
| `min_order_value` | money | `?` — no floor stated (D10) |
| `state` | enum | `?` — see state machine |
| `granted_at` | timestamp | `?` — which upstream moment (D4) |
| `expires_at` | timestamp | `?` — 30 days from *what* (D16), in which timezone (D17) |
| `reserved_for_order_id` | fk Order | `?` — spec says nothing about checkout-time reservation (D14) |
| `consumed_order_id` | fk Order | `?` |
| `consumed_at` | timestamp | `?` |

**`User`** — needs at minimum a way to answer "has this account ever completed
a purchase?" for the *first purchase* predicate. Whether that is a stored flag,
a counter, or a query over orders is `?`, and the answer to D5 changes which is
correct.

**`Order` / checkout pricing** — existing. The spec adds a discount line but
does not say where in the pricing pipeline it sits relative to promo codes,
sale prices, or other automatic discounts (D23).

### State machine(s)

**`ReferralBonus`** — drawn as far as the spec permits:

```
            [friend's first purchase "completes" — ? which moment, D4]
                              |
                              v
                          ( EARNED )
                          /    |    \
   [referrer checks out] /     |     \ [30 days elapse — ? from when, D16]
                        v      |      v
                  ( RESERVED?) |   ( EXPIRED )
                    /     \    |
      [order paid] /       \   | [checkout abandoned / payment fails]
                  v         \  v
             ( CONSUMED )    ( ? — released back to EARNED?
                              with original expiry or a new one? D26 )
```

Everything with a `?` is a transition the spec does not describe:

- `EARNED → RESERVED` — the spec says the discount is "applied automatically at
  checkout", which does not tell me whether application is atomic with payment
  or a two-phase reserve/commit. Without that I cannot design against two
  concurrent checkouts (D14).
- `RESERVED → EARNED` (release on abandonment/failure) — unspecified. If it does
  not exist, an abandoned cart silently burns the bonus.
- `CONSUMED → ?` on refund of the referrer's order — unspecified (D21).
- `EARNED|CONSUMED → REVOKED` on refund/chargeback of the *friend's* first
  purchase — unspecified (D20). If the bonus is already spent, the clawback path
  does not exist at all in this design.
- `EXPIRED` — I cannot tell whether this is a stored transition written by a job
  or a predicate evaluated at read time (D18). The two produce different
  observable histories and different notification behaviour.

**`Referral`** — I cannot draw this. The spec never says how a referral starts,
so there is no initial state and no trigger for entering it. States such as
`SENT`, `SIGNED_UP`, `QUALIFIED` would all be my invention (D1).

### Events consumed / emitted

**Consumed:**

- `?` — *some* "friend completed first purchase" signal. The spec gives me the
  business phrase, not the event: order placed, payment authorized, payment
  captured, fulfilled, or return-window-closed are five different events with
  spreads of days between them (D4). I cannot pick a subscription without this.
- No signal for referral creation exists to consume (D1).

**Emitted:**

- `?` — the spec emits nothing. A grant with no `ReferralBonusGranted` event
  means the referrer is never told they have a bonus, and no downstream system
  (email, account page, analytics) can react (D25).
- `?` — no expiry-approaching or expired event, so the "expire after 30 days"
  rule is silent to the user (D25).
- `?` — no event on consumption, so finance/analytics cannot attribute discount
  cost to the referral programme.

### Main-flow sequence

```
1. Referrer ──?──> System            : referral is created
                                       ? by what mechanism — no step exists (D1)
2. Friend  ──?──> System             : friend is attributed to referrer
                                       ? at signup, at purchase, by code (D1, D2)
3. Friend  ──> Checkout              : places first purchase
4. Checkout ──> Referral service     : "first purchase completed"
                                       ? which lifecycle moment fires this (D4)
                                       ? what counts as a "first purchase" (D5, D6)
5. Referral service                  : grant 10% bonus to referrer
                                       ? stored row or derived fact (D7)
                                       ? idempotent on redelivery — key = ? (D15)
                                       ? if referrer already holds bonuses: stack,
                                         queue, or overwrite (D12)
                                       ? expires_at = granted_at + 30d ... or
                                         referral_created_at + 30d (D16, D17)
6. Referral service ──?──> Referrer  : notification
                                       ? no event specified (D25)
7. Referrer ──> Checkout             : starts "next purchase"
8. Checkout ──> Referral service     : fetch applicable bonus
                                       ? "next purchase" = literally the next
                                         order, or the next eligible one (D11)
                                       ? concurrent checkouts (D14)
9. Checkout                          : apply 10%
                                       ? of which basis (D8), capped at ? (D9),
                                         min order ? (D10)
                                       ? precedence vs existing promos (D23)
                                       ? can the user decline / defer (D22)
10. Payment                          : succeeds
11. Referral service                 : mark consumed
                                       ? what if payment fails after step 9 (D26)
                                       ? what if expiry falls between 9 and 10 (D19)
12. (later) Friend refunds order      : ? no path (D20)
    (later) Referrer refunds order    : ? no path (D21)
```

I stopped at each `?` rather than choosing. There is no step in this sequence
between 1 and 4 that the spec actually specifies.

## Perspective questions

**1. Which entities, fields, or states did you have to invent because the spec
didn't name them?**

Everything above marked `?`. Concretely: the entire `Referral` entity and its
creation mechanism; the existence (or not) of a `ReferralBonus` row; its
`state` enum; `granted_at`, `expires_at`, `consumed_at`, `reserved_for_order_id`;
the `discount_basis`; every event, consumed and emitted. The spec names exactly
one value I can put in a field: `10`. Per the hard rules I have not adopted any
of these — each is a blocked row below.

**2. Where is each new piece of state stored, and what is its lifecycle? Is
anything in the spec ambiguous between *stored* and *derived*?**

Yes — this is the central ambiguity of the requirement. "The referring user
receives a 10% discount" can be a persisted grant with a lifecycle
(earned → reserved → consumed / expired) or a checkout-time computation over
referral history. The sentence "referral bonuses expire after 30 days" leans
toward a stored object with an expiry, because a derived value has nothing to
expire — but "expire" could equally mean "the query window is 30 days". I
cannot choose: the stored design needs a grant transaction, an expiry job,
idempotency keys and a clawback path; the derived design needs none of them and
cannot represent "already used". They are not interchangeable. Blocked as D7.

Secondarily, "has completed a first purchase" is ambiguous between a stored
flag on `User` and a derived query over `Order` — and the two disagree the
moment an order is refunded (D5, D20).

**3. What happens under concurrency — two of these at the same instant? Do
effects stack, compound, overwrite, or queue?**

Unspecified in both directions, and both are real:

- *Two grants at once*: two referred friends complete their first purchases in
  the same instant. Does the referrer end up with two bonuses (queued, one per
  future order), one 20% discount (stacked), one 19% discount (compounded
  0.9×0.9), or one bonus because the second overwrites the first? The spec's
  singular "a 10% discount on their next purchase" hints at one, but does not
  say what becomes of the second. Blocked as D12.
- *Two consumptions at once*: the referrer opens two checkouts and both reach
  payment. Without a reservation step or a conditional update on bonus state,
  the same 10% is spent twice. The spec's "applied automatically at checkout"
  describes no locking, no reservation, and no compare-and-set. Blocked as D14.

**4. What makes this idempotent if the triggering event arrives twice?**

Nothing in the spec. There is no stated uniqueness constraint — not
one-bonus-per-referral, not one-bonus-per-referee, not one-bonus-per-order. If
the "first purchase completed" signal is redelivered (at-least-once delivery,
a retry, a replay), the natural implementation grants a second bonus. I need a
dedupe key and cannot derive one: `referral_id` works only if a referral yields
at most one bonus ever, which is itself unstated. Blocked as D15.

Symmetrically, applying the discount at checkout needs to be idempotent across
checkout retries; the spec describes application as a one-shot action.

**5. What is the ordering or precedence relative to features that already
exist?**

Entirely unspecified, and this is where the 10% becomes a number I cannot
compute. Unknown: whether the bonus stacks with coupon codes, sale/markdown
prices, loyalty discounts, or other automatic promotions; if it stacks, in
which order (10% off list, or 10% off an already-discounted subtotal); whether
it is pre- or post-tax and pre- or post-shipping; whether any existing "best
discount wins" rule pre-empts it. Blocked as D8 and D23. Also unspecified:
whether the "next purchase" surface includes non-web channels, guest checkout,
subscription renewals, and gift-card purchases (D22).

**6. Which failure modes are unspecified — partial write, timeout mid-flow,
upstream event never arrives?**

All of them.

- *Partial write on grant*: friend's order is committed but the bonus write
  fails. No reconciliation, backfill, or outbox is described — the referrer
  silently never gets the bonus, and nothing detects it. (D26)
- *Timeout mid-checkout*: the discount is applied, then payment fails or the
  cart is abandoned. Whether the bonus is released, and with what expiry, is
  undefined — the state machine has no edge for it. (D26)
- *Expiry race*: the bonus is valid when the discount is applied and expired by
  the time payment captures. Which timestamp governs is undefined. (D19)
- *Upstream event never arrives*: if the "first purchase" signal is lost, there
  is no periodic sweep to reconcile referrals against orders. (D26)
- *Reversal*: the friend refunds or charges back the qualifying first purchase.
  No revoke or clawback path exists, and if the referrer already spent the
  bonus there is no defined remedy at all. (D20)
- *Referrer account deleted, deactivated, or banned* between grant and
  redemption. Undefined. (D24)

## Blocked rows

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|
| D1 | "When a user refers a friend" | By what mechanism is a referral created and attributed — a shareable code, a unique link, an emailed invite, a contact import? Nothing in the spec creates a `Referral`. | The entire `Referral` entity, its state machine, its creation trigger, and steps 1–2 of the main sequence. | |
| D2 | "a friend" | Must the referee be a brand-new account? What if the friend already has an account, or already has purchase history? Can one referee be attributed to two referrers, and if so who wins — first touch or last touch? | Attribution rule; uniqueness constraint on `Referral.referee_user_id`; eligibility check at grant time. | |
| D3 | "refers a friend" | Are self-referrals, same-household, same-device, or same-payment-method referrals excluded? The spec states no anti-abuse rule. | Whether a fraud/eligibility check exists in the grant path at all, and what data it needs. | |
| D4 | "completes their first purchase" | Which lifecycle moment is "completes" — order placed, payment authorized, payment captured, order fulfilled, or return window elapsed? These are days apart. | Which upstream event the referral service subscribes to; `granted_at`; and the start of the 30-day clock if it runs from the grant. | |
| D5 | "their first purchase" | First purchase by what definition — first ever order on the account, first *paid* order, first non-refunded order? Does a zero-value, gift-card-funded, or fully-discounted order qualify? | The eligibility predicate, and whether "has purchased" is a stored flag or a derived query. | |
| D6 | "their first purchase" | If the friend's first purchase happened *before* the referral was created, does it qualify retroactively? | Whether the grant path is triggered by the purchase event or by the referral event; backfill behaviour. | |
| D7 | "the referring user receives a 10% discount" | Is the bonus a stored, addressable object with its own lifecycle, or a value derived at checkout from referral history? | Whether `ReferralBonus` exists as an entity, and with it the grant transaction, expiry job, idempotency key, and clawback path — or none of them. | |
| D8 | "a 10% discount" | 10% of what basis — item subtotal, subtotal plus shipping, tax-inclusive total, or the total after other discounts have already been applied? Is the discount itself taxable? | The pricing calculation; `discount_basis`; tax treatment; refund arithmetic. | |
| D9 | "a 10% discount" | Is there a maximum discount amount? 10% of a very large order is unbounded liability. | `max_discount_amount`; finance exposure controls. | |
| D10 | "a 10% discount" | Is there a minimum order value below which the bonus does not apply? | `min_order_value`; whether an ineligible order consumes the bonus (see D11). | |
| D11 | "on their next purchase" | Is this literally the next order the referrer places, or the next order on which the discount can validly apply? If the next order is ineligible (refunded, gift card, below minimum, different channel), is the bonus consumed or retained? | The consumption rule and the `RESERVED → CONSUMED` vs `→ EARNED` transitions. | |
| D12 | "receives a 10% discount" (singular) | If a referrer earns several bonuses, do they stack on one order (10+10 = 20%), compound (0.9 × 0.9 = 19%), queue one per future order, or overwrite each other? | Whether bonuses are a set or a single slot; the concurrency behaviour in question 3; the pricing calculation. | |
| D13 | "When a user refers a friend" | Is there a cap on how many bonuses one referrer can earn — per month, per programme, ever? | Grant-time limit checks; programme cost model. | |
| D14 | "applied automatically at checkout" | Two concurrent checkouts by the same referrer: is the bonus reserved at checkout start and released on abandonment, or only consumed atomically at payment? Without this the same 10% can be spent twice. | The `RESERVED` state, `reserved_for_order_id`, locking/compare-and-set strategy, and release-on-abandon transition. | |
| D15 | "the referring user receives a 10% discount" | If the "first purchase completed" event is delivered twice, what prevents a second bonus? What is the dedupe key — referral, referee, or order? | Uniqueness constraint; idempotency of the grant handler; safety under at-least-once delivery and replays. | |
| D16 | "Referral bonuses expire after 30 days" | 30 days from what — the referral being created, the friend's first purchase, the bonus being granted, or the referrer first being notified? | `expires_at`; the meaning of the `EXPIRED` state; whether the clock is on `Referral` or on `ReferralBonus`. | |
| D17 | "expire after 30 days" | Calendar days or 720 hours? In which timezone — the referrer's, the referee's, or UTC? Is day 30 inclusive, and is expiry at the instant or at end of day? | Boundary arithmetic; DST correctness; whether a bonus is valid on its 30th day. | |
| D18 | "expire after 30 days" | Is expiry a stored transition written by a scheduled job, or a predicate evaluated at read time? | Whether an expiry job exists; whether an `expired` event can be emitted; what an account page shows between the expiry instant and the job run. | |
| D19 | "applied automatically at checkout" + "expire after 30 days" | If the bonus is valid when the discount is applied but expired by the time payment captures, which timestamp governs? | Validation point in the checkout flow; the expiry race in question 6. | |
| D20 | "the friend completes their first purchase" | If the friend refunds, cancels, or charges back that first purchase, is the bonus revoked? What if the referrer has already spent it — clawback, write-off, or negative balance? | The `REVOKED` state and its transitions from both `EARNED` and `CONSUMED`; whether reversal events are consumed at all. | |
| D21 | "on their next purchase" | If the referrer refunds or cancels the order the discount was applied to, is the bonus restored? With its original expiry, or a new 30 days? Is the refund amount the discounted price or list price? | The `CONSUMED → EARNED` transition; refund arithmetic; expiry recalculation. | |
| D22 | "applied automatically at checkout" | Can the referrer see it before checkout, decline it, or save it for a larger order? Does "checkout" include guest checkout, mobile app, in-store/POS, and subscription renewals? | Whether the bonus is user-controllable state; which surfaces must integrate; the account-page contract. | |
| D23 | "applied automatically at checkout" | How does the bonus interact with existing promo codes, sale prices, loyalty discounts, and other automatic promotions — stack, mutually exclude, or best-wins? If it stacks, in what order is it applied? | Position in the pricing pipeline; the precedence question in question 5; the actual amount charged. | |
| D24 | "the referring user receives" | What happens if the referring user's account is deactivated, banned, merged, or deleted between grant and redemption? | Grant-time and redemption-time eligibility on the referrer; data retention for the bonus. | |
| D25 | (whole spec) | Is the referrer notified that they earned a bonus, or warned before it expires? The spec emits no events, so nothing downstream — email, account page, analytics, finance — can react. | Every emitted event; whether a 30-day expiry is discoverable by the user at all; discount-cost attribution. | |
| D26 | (whole spec) | What happens on partial failure — bonus write fails after the friend's order commits; payment fails after the discount is applied; the "first purchase" event is never delivered? Is there an outbox, a retry, or a reconciliation sweep? | Delivery guarantees; release-on-failure transition; whether any process detects a bonus that should exist but does not. | |
