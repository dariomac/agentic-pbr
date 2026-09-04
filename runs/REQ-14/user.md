# PBR — User — REQ-14

## Artifact: help-centre article

> Writing note: every `?` below marks a sentence I could not write, because the
> spec does not say. I have not guessed, softened, or worded around any of them.

---

### Refer a friend, get 10% off

**What do I get?**

When a friend you referred completes their first purchase, you get a **10%
discount on your next purchase**.

10% off ?

<!-- ? = the spec says "a 10% discount" but never says 10% of what. Off the
item price, the order subtotal, the total including shipping, including tax?
Is there a maximum discount amount, or a minimum order value? I cannot write
"10% off your order total" — the spec does not say that. -->

**What do I have to do to get it?**

?

<!-- ? = the spec says "when a user refers a friend", but nothing in the spec
describes how a referral happens. There is no link, no code, no invite, no
form, no email. I cannot tell a customer how to refer someone, because the
feature that does the referring is not described. This is the first sentence
of the article and I cannot write it. -->

Who counts as a friend?

?

<!-- ? = the spec does not say whether the friend must be a brand-new customer,
whether someone with an existing (dormant) account can be referred, or whether
the two people may share an address, device, or payment method. -->

**How do I know when my friend's purchase counts?**

Your bonus is created when your friend **completes their first purchase**.

?

<!-- ? = "first purchase" is not defined. First ever on the platform, or first
after you referred them? Does "completes" mean order placed, order paid, order
shipped, or the return window closed? Is there a minimum order value? These
change the day the customer's 30-day clock starts, so I cannot leave it vague. -->

**How will I know I have it?**

?

<!-- ? = nothing in the spec creates a notification, an email, a badge, an
account page, a banner, or any other way for you to learn that a bonus exists.
The spec says the discount "is applied automatically at checkout" — which means
the first moment the customer is guaranteed to see the bonus is the moment it
is being spent. I cannot write "we'll email you" or "check your account page",
because neither exists in the spec. -->

**How do I use it?**

The discount is **applied automatically at checkout** — you don't need to enter
a code.

Can I choose not to use it on this order, and save it for a bigger one?

?

<!-- ? = "automatically" and "your next purchase" together suggest the bonus is
spent on whatever you happen to buy next, even a £3 order. The spec does not
say whether the customer can decline, defer, or remove it. -->

Can I use it together with a sale price or another discount code?

?

<!-- ? = the spec says nothing about stacking, combining, or precedence against
other promotions. -->

**What if I refer more than one friend?**

?

<!-- ? = the spec describes one referral producing one bonus for "their next
purchase". It does not say whether two bonuses stack into 20%, queue up for two
separate purchases, or whether only one can be held at a time. -->

**When does it stop being available?**

Referral bonuses **expire after 30 days**.

30 days from ?

<!-- ? = the spec does not say what starts the 30 days: the moment you referred
your friend, the moment your friend completed their first purchase, or the
moment the bonus was created. These can be weeks apart. This is the single most
important sentence in an expiry article and I cannot write it. -->

Where can I see my expiry date?

?

<!-- ? = no screen, list, or account section is specified where the bonus and
its remaining time are visible. Under the spec as written, a customer can only
discover their bonus expired by *not* seeing it at a checkout. -->

**What if something goes wrong?**

My friend cancelled or refunded their first purchase — do I keep my bonus?

?

<!-- ? = the spec does not say whether the bonus is revoked, nor what happens
if it has already been spent. -->

I returned the order I used my bonus on — do I get the bonus back?

?

<!-- ? = not specified. -->

My bonus expired before I bought anything — can I get it back?

?

<!-- ? = the spec defines no recovery, extension, or support path. -->

---

### What your friend gets

?

<!-- ? = the spec describes the referring user's reward only. It does not say
whether the referred friend receives anything, is told they were referred, or
sees any part of this at all. I cannot write this section, and I cannot
truthfully write "your friend gets nothing" either — the spec is silent, not
negative. -->

---

## Perspective questions

**1. Which customer questions could I not answer at all?**

Four of the five. Only "What do I have to do to get it?" has even a partial
answer, and it is the wrong half — I can say the reward is automatic at
checkout, but I cannot say how to refer anyone in the first place.

- *What do I get, exactly?* — **partial.** "10%" is a number without a base.
  No cap, no minimum, no stacking rule.
- *What do I have to do to get it?* — **no.** The referral mechanism does not
  exist in the spec.
- *How will I know I have it?* — **no.** No notification, no visible state.
- *When does it stop being available?* — **no.** "30 days" has no start event.
- *What if something goes wrong?* — **no.** No reversal, refund, or expiry
  recovery behaviour is defined.

**2. How does the user discover this happened? Is any notification, badge, or
screen actually required by the spec, or did I assume one?**

The spec requires **none**. I assumed one and had to delete it. Re-reading the
three sentences: the only guaranteed moment of contact between the feature and
the customer is the checkout page, and even there the spec says the discount is
applied — it does not say it is *shown*, labelled, or explained. A customer
could complete a discounted checkout without ever learning why the price
changed, and could equally lose a bonus at day 31 having never been told they
had one. "Applied automatically" reads as a convenience; in a spec with no
notification requirement it is actually the *only* delivery channel, which
makes it load-bearing in a way the product team probably did not intend.

**3. Is the visibility of the state defined — can the user see it before it
matters to them?**

No. There is no defined place where a pending bonus, its value, or its expiry
date lives. The state is invisible until it is spent, at which point it is
already gone. This makes the 30-day expiry unwritable as customer-facing
documentation: I would be telling people to watch a clock they cannot see.

**4. Are all the affected parties covered?**

No. The spec involves two people and specifies one. The referred friend's
experience is **silently empty**: it is not stated whether they get a welcome
offer, whether they are told who referred them, whether they consent to being
referred, or whether being "referred" is visible to them anywhere. The friend
also carries an obligation in this feature — their first purchase is the
trigger — without any specified awareness of it.

**5. What would a user try that the spec doesn't forbid — and would that be
abuse?**

- **Referring yourself** with a second email address, then making one small
  first purchase to mint a 10% discount for the main account. Nothing in the
  spec forbids it, and with no definition of "friend" there is no rule to
  enforce.
- **Farming**: referring N throwaway accounts, each of which makes a minimal
  first purchase, if bonuses stack or queue (undefined — see U7).
- **Trigger-and-refund**: friend buys, referrer's bonus is created, friend
  refunds. Whether the bonus survives is undefined, so the cheapest read of the
  spec is that it does.
- **Bonus on a £1 order**: if "next purchase" is literal and automatic, an
  honest customer can be *harmed* by the same gap — a 10p discount consumes the
  whole bonus. That is the inverse of abuse and equally a defect.

**6. Is there anything I'd have to word vaguely to avoid promising something
the system might not do?**

Yes, and I refused to. The three sentences I would have written vaguely if I
were trying to ship this article on time:

- "We'll let you know when your bonus is ready" — invents a notification.
- "Your bonus is valid for 30 days from when it's issued" — invents an issue
  event and calls it the start of the clock.
- "You'll see your discount at checkout" — the spec says applied, not shown.

Each of those reads perfectly well and each is a fabrication. That the article
*could* be finished smoothly by writing them is the measure of how much of this
feature is undefined.

## Blocked rows

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|
| U1 | "When a user refers a friend" | How does a user actually refer someone? Link, code, invite email, form? The spec defines no referral mechanism at all. | The entire "What do I have to do to get it?" section — the first thing a customer needs. | |
| U2 | "refers a friend" | Who is eligible to be referred? Must they be a brand-new customer, or can an existing/dormant account be referred? Can referrer and friend share an address, device, or payment method? | Eligibility paragraph; also the only defence against self-referral (see U15). | |
| U3 | "the friend completes their first purchase" | "First" ever, or first after the referral? "Completes" = placed, paid, shipped, or return-window closed? Is there a minimum order value? | "How do I know when my friend's purchase counts?" and the start of the 30-day clock. | |
| U4 | "a 10% discount" | 10% of what — item price, subtotal, total including shipping, including tax? Is there a maximum discount or a minimum order value? | "What do I get, exactly?" — the headline claim of the article. | |
| U5 | "on their next purchase" | Is it literally the next transaction (so a £3 order burns the bonus), or the next purchase the customer chooses to apply it to? | Whether the article warns customers to save the bonus for a large order. | |
| U6 | "applied automatically at checkout" | Can the customer decline, remove, or defer it? Can it combine with sale prices or other discount codes, and which wins? | "How do I use it?"; stacking is the single most-asked support question for discounts. | |
| U7 | "the referring user receives a 10% discount" | If a user refers several friends, do bonuses stack (20%?), queue for separate purchases, or is only one held at a time? | "What if I refer more than one friend?" — and the abuse ceiling. | |
| U8 | "Referral bonuses expire after 30 days" | 30 days from what? The referral being sent, the friend's first purchase, or bonus creation? These can be weeks apart. | "When does it stop being available?" — unwritable without a start event. | |
| U9 | "Referral bonuses expire after 30 days" | Where can a customer see a pending bonus, its value, and its expiry date before checkout? No screen, list, or account section is specified. | Any instruction to "use it before it expires"; the customer is asked to watch an invisible clock. | |
| U10 | (whole spec) | Is the referring user notified that a bonus exists? No email, push, badge, or banner is required anywhere in the spec. | "How will I know I have it?" — currently the customer's only discovery moment is spending it. | |
| U11 | "The discount is applied automatically at checkout" | Is the discount *shown and labelled* at checkout, or merely applied to the price? "Applied" does not imply visible. | Whether the customer can tell a referral bonus from a sale price; also whether U10 has any fallback at all. | |
| U12 | "When a user refers a friend" | What does the referred friend experience? Do they get an offer, a notice that they were referred, or any visibility? The second party is specified only as a trigger. | The friend-facing section of the article, and any invite copy the friend receives. | |
| U13 | "the friend completes their first purchase" | If the friend cancels or refunds that first purchase, is the referrer's bonus revoked? What if it has already been spent? | "What if something goes wrong?"; also the cheapest abuse route (see U15). | |
| U14 | "a 10% discount on their next purchase" | If the referrer returns or cancels the order the bonus was applied to, is the bonus restored, or consumed? | "What if something goes wrong?" — returns are routine, not an edge case. | |
| U15 | "When a user refers a friend" | Nothing forbids referring yourself via a second account, or farming many throwaway accounts each making a minimal first purchase. Is either intended to be prevented? | Terms/abuse section; without U2 and U7 there is no rule to state or enforce. | |
| U16 | "Referral bonuses expire after 30 days" | Is there any recovery — extension, reinstatement, or support override — for a bonus that expired unused (particularly given the customer was never told it existed)? | "My bonus expired — can I get it back?", which support will be asked daily. | |

**Blocked rows: 16.**
