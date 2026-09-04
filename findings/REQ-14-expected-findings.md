# REQ-14 — expected findings

Reference baseline. Use it to check whether a run of the perspectives is still
working — after you edit a perspective, swap a model, or hand the technique to
someone new. A run that reproduces these rows is healthy; a run that misses
several is a regression worth investigating.

Note what every single row has in common: **not one of these is a coding defect.**
No amount of unit testing, mutation testing or coverage finds any of them.

## Tester

| # | Spec text | Question | Blocks |
|---|-----------|----------|--------|
| T1 | "a 10% discount" | 10% of what — subtotal, or total including tax and shipping? | Any expected value in any test |
| T2 | "a 10% discount" | Rounding rule and currency precision? | Boundary cases |
| T3 | "a 10% discount" | Is there a maximum? 10% of a $50,000 order is $5,000. | Risk cases |
| T4 | "their next purchase" | Next after B's purchase, or after the bonus is granted? What about a cart already in flight? | Sequencing cases |
| T5 | "expire after 30 days" | 30 days from which event? | Expiry cases |
| T6 | "expire after 30 days" | Which timezone, and is day 30 inclusive? | Boundary cases |
| T7 | "completes their first purchase" | What counts as "completes" — payment authorised, captured, shipped, or past the returns window? | The trigger itself |
| T8 | (absent) | B refunds after A has redeemed. Clawback, absorb, or block? | Reversal cases |
| T9 | "applied automatically at checkout" | Precedence against other discounts and coupons? | Interaction cases |

## Designer

| # | Spec text | Question | Blocks |
|---|-----------|----------|--------|
| D1 | "receives a 10% discount" | Stored credit or rule evaluated at checkout? | Schema, and everything downstream of it |
| D2 | (absent) | There is no referral entity. What are its states and transitions? | The whole state machine |
| D3 | (absent) | Two referrals complete simultaneously — stack, compound, overwrite, or queue? | Concurrency design |
| D4 | "completes their first purchase" | The purchase-completed event is delivered twice. What makes granting idempotent? | Event handling |
| D5 | "expire after 30 days" | Expiry by scheduled job or evaluated on read? Different failure modes. | Expiry design |
| D6 | (absent) | Bonus granted but the notification send fails — is the grant still valid? | Failure design |

## User

| # | Spec text | Question | Blocks |
|---|-----------|----------|--------|
| U1 | (absent) | How does A find out they have a bonus? Nothing requires any notification. | "How will I know?" |
| U2 | (absent) | Can A see the bonus before checkout, or does it just appear? | "Where do I find it?" |
| U3 | "the friend" | What does B get? Nothing? Deliberate, or an omission? | Half the readership of the article |
| U4 | (absent) | Can A refer themselves from a second account? Spec is silent on abuse. | Terms and limits |
| U5 | "expire after 30 days" | Is A warned before expiry? | "When does it end?" |
| U6 | "a 10% discount" | I cannot tell a customer what they'll save, because T1 is undefined. | The single sentence customers care about most |

## Why the baseline is worth keeping

Take REQ-14 to any coding agent and ask for the Gherkin. You will get clean,
readable, internally consistent scenarios — with a **silent decision on every
row above.** Then the inner loop turns green, coverage looks great, and the
mutation score is fine.

**Green suite. Wrong product.** That gap is what this repository exists to
close, and this file is how we tell whether we are still closing it.
