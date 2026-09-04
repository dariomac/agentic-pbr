# agentic-pbr

Perspective-Based Reading for specifications, run by agents.

Install it as a Claude Code plugin and run `/apbr:run` against a requirement.
Three readers take a role each, try to do that role's real job with your spec,
and hand back every point where the spec didn't let them finish. Each of those
is a question only a human can answer.

## The one-sentence version

Your pipeline can prove the code matches the spec. It can never tell you the
spec was right. So the spec is the last artifact a human has to read — and
reading it well is a technique, not a matter of paying attention.

## The technique

**Perspective-Based Reading** (Basili, Green, Laitenberger, Lanubile, Shull,
Sørumgård & Zelkowitz, with NASA/GSFC, 1994–95). You don't *review* the
requirement. You take a role, **produce the artifact that role would produce**,
and report everywhere you couldn't finish.

That inversion is the whole trick:

> Reading lets you skim past a gap. Producing makes the gap physical.

You can read "a 10% discount" and nod. You cannot write `expected_total = ?`
and nod.

Three perspectives ship by default:

| Perspective | Produces | Catches |
|---|---|---|
| **Tester** | The acceptance test cases | Undefined values, missing boundaries, untestable claims |
| **Designer** | The high-level design | Missing entities and states, concurrency, idempotency, failure modes |
| **User** | The help-centre article | Nothing that tells the customer what happened; unspecified second parties |

Eight more — security, SRE, fraud, i18n and others — are sketched in
`perspectives/EXTRA-perspectives.md`, along with a template for writing your own.

## Install

This repository is a Claude Code plugin *and* its own marketplace:

```
/plugin marketplace add dariomac/agentic-pbr
/plugin install apbr@agentic-pbr
```

## Run it

In any project:

```
/apbr:run path/to/REQ-14.md
```

The first argument is a spec file **or** a directory to walk. The optional
second is where runs are written, defaulting to `./pbr-runs`:

```
/apbr:run docs/specs/ ./pbr-runs
```

You get one file per perspective plus a `consolidated.md` that groups every
blocked row by the spec clause it attacks, and marks which clauses two or three
readers hit independently. A directory run also gets a `summary.md` so the
weakest specs are visible at a glance.

Nothing is copied into your project — the perspectives live in the plugin, so
updating the plugin updates the technique. Your specs are never modified.

**Try it on a bad requirement first.** Three sentences is enough:

```markdown
When a user refers a friend and the friend completes their first purchase, the
referring user receives a 10% discount on their next purchase. The discount is
applied automatically at checkout. Referral bonuses expire after 30 days.
```

That spec produces well over a dozen findings, and **not one of them is a coding
defect.** 10% of what — subtotal, or total including tax and shipping? Thirty
days from which event, in whose timezone? What happens when the friend refunds
after the discount is redeemed? No amount of unit testing, mutation testing or
coverage finds any of it.

## What to do with the output

Every row is a question, and the Decision column is deliberately empty.

> **Agents find ambiguity. Humans resolve it.**

Finding scales. Resolving doesn't. Every prompt in this plugin ends with the
same hard rules — never resolve an ambiguity, never invent a value, never fill
in the Decision column — because the moment the agent answers its own questions
you're back to the pipeline grading its own homework, and your confidence goes
up while your correctness doesn't.

**Delegable** — checks internal to the document: ambiguity, testability,
contradiction, missing cases, terminology drift, traceability.

**Not delegable** — anything needing knowledge the model doesn't have: what the
business actually wants, what the regulator requires, what you tried in 2023
that opened a fraud hole, which of two stakeholders wins.

An agent can tell you "10% of what?" is undefined. Only you can decide the
answer, and deciding *is* the engineering.

So: fill in the Decision column yourself. Then write the Gherkin.

## Where it goes in the loop

A PBR gate sits **in front of** the outer BDD loop. A blocked row is a **red at
the spec level** — Red-Green-Refactor applied to the spec, before a line of
Gherkin exists.

And it's measurable, which matters: IEEE 610.12-1990 defined software
engineering as a *systematic, disciplined, **quantifiable*** approach. Blocked
rows per requirement, per perspective; ambiguity density; share of Gherkin
traceable to a specific spec clause.

## Honest about the evidence

PBR is well studied, not settled. The original experiments had small samples,
ran inside the NASA SEL environment, and showed clearer gains at team level than
individual level. A later paper is titled *"Are the Perspectives Really
Different?"* and challenges the central assumption. Read it before presenting
this as proven.

- [The Empirical Investigation of Perspective-Based Reading (PDF)](https://www.cs.umd.edu/~mvz/handouts/emp_pbr.pdf)
- [Journal version](https://link.springer.com/article/10.1007/BF00368702)
- [Are the Perspectives Really Different?](https://link.springer.com/article/10.1023/A:1009848320066)
- [How Perspective-Based Reading Can Improve Requirements Inspections](https://www.researchgate.net/publication/2955334_How_Perspective-Based_Reading_Can_Improve_Requirements_Inspections)

Nothing above has been re-validated for the agent-run variant. That is part of
what this repository is for.

## Why this was abandoned, and why that changed

Inspections didn't fall out of use because they didn't work. They fell out of
use because they cost several trained readers and a meeting per requirement.
That cost is gone.

**AI didn't make the fundamentals obsolete. It made them affordable.**

---

Improving a perspective, adding a new one, or working on the plugin itself:
see [CONTRIBUTING.md](CONTRIBUTING.md). How blindness and the plugin root
actually work: [docs/design-notes.md](docs/design-notes.md).
