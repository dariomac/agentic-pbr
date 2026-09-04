---
description: Run a blind Perspective-Based Reading pass (tester, designer, user) over a spec file or a directory of specs
argument-hint: <spec-file-or-dir> [output-dir]
---

Run a Perspective-Based Reading pass.

- **Spec target:** `$1` — a single spec file, or a directory to walk.
- **Output directory:** `$2` — defaults to `./pbr-runs` if not given.

## 1. Resolve the spec list

If `$1` is a file, the list is that one file.

If `$1` is a directory, walk it for `.md` files, recursively. Then **stop and
report the list before spawning anything.** State how many specs you found and
how many subagents that implies (three per spec). If it is more than 5 specs
(15 subagents), ask the user to confirm before proceeding — a careless
`/apbr:run docs/` should not quietly become a hundred agents.

Derive `<SPEC-ID>` from the spec's path relative to `$1`, slugified — so
`auth/REQ-14.md` becomes `auth-REQ-14`, not `REQ-14`. Two specs in different
folders can share a filename; their outputs must not collide.

## 2. Spawn the readers

For each spec, create `<output-dir>/<SPEC-ID>/` and spawn the three perspective
subagents **in a single message, in parallel**: `pbr-tester`, `pbr-designer`,
`pbr-user`.

Each prompt contains **only**:

- the spec path,
- the output path `<output-dir>/<SPEC-ID>/<perspective>.md`,
- the reminder that it is running blind.

**Do not** put the spec text, another perspective's output, a hint, an example
finding, or anything from a `findings/` directory into a perspective prompt.
Blindness is the mechanism: readers who see each other converge, and a converged
pass finds the defects one reader would have found alone.

Never write outside `<output-dir>`. The spec files themselves are read-only —
this command never edits a spec, however tempting the fix looks.

### While they run

The readers are tracked work: you are re-invoked automatically as each one
finishes. So do nothing while you wait. Do not poll them, do not re-read their
output paths hoping a file has appeared, and do not schedule a wake-up — a timer
is not what tells you they are done, and scheduling one is both unnecessary and
a common source of malformed tool calls.

Report each completion in one line (`<perspective> finished, N blocked rows`),
and say what you are still waiting on. Nothing else until all three return.

## 3. Consolidate

Read the three files per spec, then write `<output-dir>/<SPEC-ID>/consolidated.md`:

- every blocked row from all three perspectives, keeping its original id,
- rows grouped by the spec clause they attack, with cross-perspective
  corroboration marked (a clause hit by two or three roles independently is the
  strongest signal in the whole technique),
- rows only one perspective found, called out as the value the parallel run
  bought,
- the metrics: blocked rows per perspective, distinct spec clauses implicated,
  ambiguity density.

For a directory run, also write `<output-dir>/summary.md`: one row per spec with
its blocked-row counts, so the weakest specs are visible at a glance.

## 4. Your own hard rules as the orchestrator

You are an agent too. The boundary applies to you:

1. **Never fill in the Decision column.** Not "obviously it's the subtotal", not
   a recommendation dressed as a default.
2. Do not merge two rows that ask different questions just to shorten the table.
3. Hand the open questions back to the human and stop. Answering them yourself
   is the pipeline grading its own homework: confidence goes up, correctness
   does not.

Report the consolidated table to the user and ask them to fill in the decisions.
