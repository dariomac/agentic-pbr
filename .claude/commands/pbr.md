---
description: Run a blind Perspective-Based Reading pass (tester, designer, user) over a spec
argument-hint: <path-to-spec> [extra perspective names]
---

Run a Perspective-Based Reading pass over the spec at: $1

## How to run it

1. Derive `<SPEC-ID>` from the spec filename (e.g. `specs/REQ-14.md` → `REQ-14`).
2. Create `runs/<SPEC-ID>/`.
3. Spawn the three perspective subagents **in a single message, in parallel**:
   `pbr-tester`, `pbr-designer`, `pbr-user`.

Each prompt contains **only**:

- the spec path,
- the output path `runs/<SPEC-ID>/<perspective>.md`,
- the reminder that it is running blind.

**Do not** put the spec text, another perspective's output, a hint, an example
finding, or anything from `findings/` into a perspective prompt. Blindness is
the mechanism: readers who see each other converge, and a converged pass finds
the defects one reader would have found alone.

## After they return

Read the three files, then write `runs/<SPEC-ID>/consolidated.md`:

- every blocked row from all three perspectives, keeping its original id,
- rows grouped by the spec clause they attack, with cross-perspective
  corroboration marked (a clause hit by two or three roles independently is the
  strongest signal in the whole technique),
- rows only one perspective found, called out as the value the parallel run
  bought,
- the metrics: blocked rows per perspective, distinct spec clauses implicated,
  ambiguity density.

## Your own hard rules as the orchestrator

You are an agent too. The boundary in `README.md` applies to you:

1. **Never fill in the Decision column.** Not "obviously it's the subtotal", not
   a recommendation dressed as a default.
2. Do not merge two rows that ask different questions just to shorten the table.
3. Hand the open questions back to the human and stop. Answering them yourself
   is the pipeline grading its own homework: confidence goes up, correctness
   does not.

Report the consolidated table to the user and ask them to fill in the decisions.
