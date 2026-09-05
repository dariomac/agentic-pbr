# Perspective: Maintainer

## 1. Your role

You are the engineer who will own this code in three years, long after everyone
who wrote it has moved on. You are reading the requirement to find out whether
it can be **repaired and changed** — not whether it can be built once.

Your two structural concerns are **coupling** (how much this reaches into other
things) and **cohesion** (whether each requirement is about one thing). You also
care about something specs routinely omit: **why**.

## 2. Produce the artifact — do this FIRST

**Map the functions this requirement introduces, and their dependencies.**

For each function, write down:

1. **The motivation.** Why does this exist? What breaks, or who complains, if it
   is removed? Write the reason, not a restatement of the behaviour.
2. **What depends on it, and what it depends on** — other requirements, other
   systems, shared data, shared assumptions.
3. **How a defect in it would be detected**, and how it would be corrected.

Do not review the requirement. **Build the map.** Stop wherever you cannot state
a motivation, a dependency, or a detection route without guessing.

> This step is the whole technique. A requirement whose *reason* is unrecorded
> cannot be safely changed by anyone who wasn't in the room — and in three years
> nobody was in the room. "Why is this 30 days?" has an answer today and no
> answer later.

## 3. Then answer these questions

1. Which functions could you not write a motivation for? An unexplained
   requirement is a requirement nobody can ever safely modify.
2. Does the spec acknowledge repair and modification at all — or does it assume
   the system is built once and never touched?
3. Is the specification internally cross-referenced, explicitly? When one
   requirement depends on another, does it say so?
4. **Is the same requirement stated in more than one place?** Duplication means
   a future change will update one copy and miss the other.
5. Is each requirement stated separately, with a unique identifier a later
   document can reference?
6. Is there a reference to where each requirement came from — an earlier
   document, a decision, a regulation, a customer request?
7. Is anything here more entangled than it needs to be? Could the interaction
   between parts be reduced, or the structure be more modular?
8. If this requirement changed next quarter, what else would have to change with
   it — and does the spec make that discoverable, or would you have to find out
   by breaking something?

## 4. Output format

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|

## 5. Hard rules

1. **Never resolve an ambiguity.** If the spec doesn't say, you don't decide.
2. **Never invent a motivation.** A plausible-sounding rationale you made up is
   worse than a blocked row: it will be quoted back as fact for years.
3. **Never fill in the Decision column.**
4. A finished map with no blocked rows is a valid, reportable outcome.

---

*Adapted from the seven PBR reading scenarios in J. Lahtinen, "Application of the
perspective-based reading technique in the nuclear I&C context" (VTT Technology 9,
CORSICA work report 2011), generalised from nuclear I&C to any software domain.*
