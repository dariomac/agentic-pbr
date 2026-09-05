---
description: Run a blind Perspective-Based Reading pass over a spec file or a directory of specs, using a selectable set of perspectives (tester, designer, user, maintainer, verifier, regulator, contractor)
argument-hint: <spec-file-or-dir> [output-dir] [perspectives]
---

Run a Perspective-Based Reading pass over these arguments:

```
$ARGUMENTS
```

Split that line on whitespace and read it positionally. Do not use any other
source for these values, and do not infer them from the surrounding
conversation:

1. **Spec target** (required) — a single spec file, or a directory to walk.
2. **Output directory** (optional) — defaults to `./pbr-runs`.
3. **Perspectives** (optional) — a comma-separated list, defaulting to
   `tester,designer,user`. It is the only argument that contains no `/` or `.`,
   which is how you tell it apart from a path if you are ever unsure.

If the line is empty, say what the command expects and stop. If a value is
present but you cannot tell which slot it belongs to, ask — never guess at a
spec path or an output directory, because guessing wrong either reads the wrong
document or writes into the wrong project.

Echo the three resolved values back before doing anything else, so a
misreading is visible immediately.

## 0. Choose the perspectives

Seven are available:

| Name | Produces | Reads for |
|---|---|---|
| `tester` | Acceptance test cases | Values you cannot fill in; untestable claims |
| `designer` | High-level design and data flow | Missing entities, states, interfaces, constraints |
| `user` | Help-centre article | What the customer is never told; unspecified parties |
| `maintainer` 🧪 | Function and dependency map | Unrecorded rationale, coupling, duplication, traceability |
| `verifier` 🧪 | Negative requirements and failure analysis | Behaviour the spec forgot to forbid; correlated failure |
| `regulator` 🧪 | Obligations list and structural map | Standards, references, internal inconsistency, evidence |
| `contractor` 🧪 | Assumed-knowledge list and alternative readings | Tribal knowledge; requirements an outsider would misread |

🧪 marks an **experimental** perspective: adapted from published scenarios, but
not yet validated against enough real specifications and carrying no reference
baseline. They are never in the default set. If you suggest one, or run one
because the user asked, say once that it is experimental — do not present its
findings as carrying the same weight as the stable three, and never quietly
promote one into a default run.

**Do not run all seven by default.** More readers is not better: the cost is
linear, the overlap grows, and the source research is explicit that only the
most relevant perspectives should be selected for a given inspection. Keep
`tester` and `designer` in almost every pass — they are the fundamental two.

If the user named perspectives in the third argument, use exactly those. Otherwise use the
default three, and — only if the spec obviously invites it — say in one line
which additional perspective looks worth a second pass, and why. Do not run it
uninvited.

Reject an unknown name rather than guessing at what was meant.

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

For each spec, create `<output-dir>/<SPEC-ID>/` and spawn the selected
perspective subagents **in a single message, in parallel** — `pbr-<name>` for
each name chosen in step 0.

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

**Resolve a relative `<output-dir>` against the working directory of this
session — the project the user is running in — and never against the plugin's
own directory.** Those are different places, and the plugin's directory may be a
read-only cache or, when the plugin is sideloaded from a checkout, somebody's
working repository. Writing run output there pollutes it. Before the first
write, state the absolute path you are writing to, so a wrong root is visible
immediately rather than after the run. Pass each subagent an absolute output
path for the same reason.

### While they run

The readers are tracked work: you are re-invoked automatically as each one
finishes. So do nothing while you wait. Do not poll them, do not re-read their
output paths hoping a file has appeared, and do not schedule a wake-up — a timer
is not what tells you they are done, and scheduling one is both unnecessary and
a common source of malformed tool calls.

Report each completion in one line (`<perspective> finished, N blocked rows`),
and say what you are still waiting on. Nothing else until they all return.

## 3. Consolidate

Read the perspective files for each spec, then write
`<output-dir>/<SPEC-ID>/consolidated.md`:

- every blocked row from all three perspectives, keeping its original id,
- rows grouped by the spec clause they attack, with cross-perspective
  corroboration marked (a clause hit by two or more roles independently is the
  strongest signal in the whole technique — state it as a fraction of the
  perspectives actually run, e.g. 3/4, never out of a number you did not run),
- rows only one perspective found, called out as the value the parallel run
  bought,
- the metrics: which perspectives were run, blocked rows per perspective,
  distinct spec clauses implicated, ambiguity density.

### Separate what the spec already admitted from what the readers found

Before counting anything, scan the spec for gaps it declares about itself: a
"Known gaps", "Open questions" or "Open decisions" section, and inline markers
like `TBD`, `TODO`, `underspecified`, `not specified`, `unresolved`, `to be
confirmed`, or a reference to a numbered open decision.

Then mark every blocked row **pre-announced** or **discovered**, and report the
two counts separately.

This is not bookkeeping — it is what makes the corroboration number mean
anything. Three readers agreeing on a clause the spec explicitly flags is the
expected outcome, not a signal: the document sent all three to the same place.
Corroboration is evidence of independent discovery **only on clauses the spec
did not pre-announce.** Say so plainly in the consolidated file, and do not
report a headline finding that rests on a pre-announced gap without labelling
it as one.

A pre-announced row still has value when it goes further than the announcement
— naming a consequence, a scope, or an owner the spec never mentions. Mark
those **extended**, and say what they added.

### Do not claim independence you cannot verify

The readers read the whole spec, including any section listing its own gaps.
Never write that a perspective did not see something that is in the spec. You
did not observe the readers' reasoning; you only have their output files. State
what the files show and nothing more.

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
