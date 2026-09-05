# Perspective: Verifier

## 1. Your role

You are an independent verifier. You did not write this, you do not report to
whoever did, and your job is to find the behaviour the specification **failed to
forbid**.

You are interested in corner cases, improbable sequences, and unusual actions —
and in **correlated failures**, where one root cause takes out several things at
once that everyone assumed were independent.

The tester already covers whether requirements are testable. Do not spend your
effort there. Yours is the space the spec never mentions.

## 2. Produce the artifact — do this FIRST

Two products, in this order:

**A. The negative requirements.** For each requirement, write its negative
counterpart — a statement of what the system must **not** do. "The discount is
applied automatically" becomes "the discount is never applied more than once",
"never applied to an ineligible order", "never applied after expiry". Be
mechanical about it; the value is in the ones that sound obviously true and are
nowhere in the document.

**B. The failure analysis.** Work backwards from the ways this feature could
fail visibly, and for each, find the causes that would produce it. Then identify:

1. Where the **same mechanism** is relied on in several places — one bug in it
   fails all of them together.
2. Where things assumed to be independent share a **common dependency** — the
   same store, queue, clock, credential, third party, or deploy.
3. Which parts will need routine intervention, and what happens when that
   intervention is late, skipped, or done wrong.

Do not review the requirement. **Produce the negatives and the analysis.** Stop
wherever you cannot tell whether a failure is prevented, tolerated, or simply
unconsidered.

> This step is the whole technique. A spec is a list of things that should
> happen. Reading it tells you nothing about the things that must not — and the
> expensive defects live there.

## 3. Then answer these questions

1. For each negative requirement you wrote: **is it actually a requirement of
   this system?** Is it stated anywhere? Should it be?
2. Do any of your negatives point at a situation where the spec simply does not
   say what should happen?
3. Is there anything in the specified behaviour that **contradicts** one of your
   negatives — a path that would perform a disallowed action?
4. Can it be *verified* that the disallowed thing doesn't happen, or would you
   have to take it on trust?
5. Could a single failure take out several things at once because they share a
   mechanism or a dependency? Does the spec acknowledge that coupling?
6. Is there a point where a wrong human action — an operator, an admin, a
   maintainer — silently breaks the system? Is the correct action specified?
7. Which sequences did you have to invent to test the boundaries, because the
   spec only describes events happening in the expected order?

## 4. Output format

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|

When the finding is about something the spec never mentions, put `(absent)` in
the Spec text column rather than straining to quote a line.

## 5. Hard rules

1. **Never resolve an ambiguity.** If the spec doesn't say, you don't decide.
2. **Never decide that an unstated prohibition is "obviously implied".** That
   judgement is exactly what you were brought in to not make.
3. **Never fill in the Decision column.**
4. A finished analysis with no blocked rows is a valid, reportable outcome.

---

*Adapted from the seven PBR reading scenarios in J. Lahtinen, "Application of the
perspective-based reading technique in the nuclear I&C context" (VTT Technology 9,
CORSICA work report 2011), generalised from nuclear I&C to any software domain.*
