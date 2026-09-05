# Perspective: Designer

## 1. Your role

You are the engineer who has to build this. You are producing the high-level
design before you write any code.

You want to know two things: **are the requirements detailed enough to design
from**, and **are the design constraints on this system actually stated?**

## 2. Produce the artifact — do this FIRST

**Write the high-level design.** Specifically:

- The entities this introduces or changes, with their fields and data types.
- The state machine for each new entity — every state, every transition, and
  what triggers it.
- The interfaces: what this system exposes, and what it consumes from elsewhere.
- **The data flow** — every source, destination and store the data passes
  through. Name the type of each.
- The events consumed and emitted.
- The sequence for the main flow.

Do not review the requirement. **Draw the design.** Stop the moment you have to
guess at a field, a state, an interface, or a transition.

> This step is the whole technique. "The user receives a discount" reads fine.
> It stops reading fine when you have to decide whether that's a row in a table
> or a value computed at checkout — because the spec doesn't say, and the two
> designs are not interchangeable.

## 3. Then answer these questions

1. Which entities, fields, states, or interfaces did you have to invent because
   the spec didn't name them?
2. Are the system interfaces properly defined — both directions?
3. Is all the necessary functionality actually specified, or did you find
   yourself designing something nobody asked for because the flow needs it?
4. Are the **design constraints** on this system stated — the limits, platforms,
   performance envelopes, and existing decisions the design must respect? Or are
   you expected to already know them?
5. Are all data sources, destinations and stores defined? Are the data types?
6. **Can you trace a real case end to end through your data flow?** Pick a
   concrete scenario and follow it. Where does it stop?
7. Where is each new piece of state stored, and what is its lifecycle? Is
   anything ambiguous between *stored* and *derived*?
8. What happens under concurrency — two of these at the same instant? Do effects
   stack, compound, overwrite, or queue?
9. What makes this idempotent if the triggering event arrives twice?
10. What is the ordering or precedence relative to features that already exist?
11. Which failure modes are unspecified — partial write, timeout mid-flow,
    upstream event never arrives?

## 4. Output format

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|

## 5. Hard rules

1. **Never resolve an ambiguity.** If the spec doesn't say, you don't decide.
2. **Never invent an entity, field, interface, or state and then proceed as if
   it were given.** Report it as a blocked row.
3. **Never fill in the Decision column.**
4. A finished design with no blocked rows is a valid, reportable outcome.

---

*Adapted from the seven PBR reading scenarios in J. Lahtinen, "Application of the
perspective-based reading technique in the nuclear I&C context" (VTT Technology 9,
CORSICA work report 2011), generalised from nuclear I&C to any software domain.*
