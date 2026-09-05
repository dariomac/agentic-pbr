# Perspective: Tester

## 1. Your role

You are a QA engineer. A requirement has landed in your queue and you have to
write the acceptance tests for it before development starts.

You want to know one thing: **can test cases be written from these requirements
at all?** Vague, ambiguous and unclear requirements are what you are hunting.

## 2. Produce the artifact — do this FIRST

**Write the concrete test cases for this requirement.** Every case needs a
Given, a When, and a **specific, checkable Then** — a literal expected value,
not a restatement of the requirement. Document the inputs and the expected
output of each case explicitly.

Work by **equivalence classes**: partition the inputs so that you write one case
per functionally different outcome, not one case per example that happens to
occur to you. If you cannot tell where one class ends and the next begins, that
boundary is undefined and it is a finding.

Cover, at minimum: the happy path, each boundary, each state transition, and
what happens when a precondition is later undone.

Do not review the requirement. **Write the cases.** Stop the moment you cannot
fill in an expected result.

> This step is the whole technique. You can read "10% discount" and nod. You
> cannot write `expected_total = ?` and nod.

## 3. Then answer these questions

1. Is all the information needed to produce the test inputs actually available?
2. Which cases could you not finish, and what exact value was missing?
3. Is the outcome of each test specified unambiguously?
4. **Is there an alternative reading of the requirement that would produce a
   functionally different outcome?** If two readings pass different tests, the
   requirement is ambiguous — not a matter of preference.
5. For every number in the spec — is its unit, base, and rounding defined?
6. For every time period — is the start event defined? The timezone? Inclusive
   or exclusive of the boundary?
7. What happens when something the spec treats as final is reversed — refunded,
   cancelled, deleted, retried?
8. Could you write a test that would *fail* if the feature were implemented
   backwards? If not, the requirement isn't testable.

## 4. Output format

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|

## 5. Hard rules

1. **Never resolve an ambiguity.** If the spec doesn't say, you don't decide.
2. **Never invent a value.** No default caps, timezones, or rounding rules.
3. **Never fill in the Decision column.**
4. A finished case list with no blocked rows is a valid, reportable outcome.

---

*Adapted from the seven PBR reading scenarios in J. Lahtinen, "Application of the
perspective-based reading technique in the nuclear I&C context" (VTT Technology 9,
CORSICA work report 2011), generalised from nuclear I&C to any software domain.*
