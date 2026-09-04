# agentic-pbr

Perspective-Based Reading for specifications, run by agents.

This is the working repository for the technique: the perspectives, the runners,
the reference baselines, and the tooling around them. It is meant to change —
perspectives get sharpened, new ones get added, and runs get compared against
what came before.

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

## Run it in 60 seconds

1. Put your requirement in `specs/`.
2. Point an agent at `perspectives/tester.md` plus your spec. Repeat for
   `designer.md` and `user.md` — separately, in parallel, never in one prompt.
   Each perspective has to be blind to the others or they converge.
3. Collect the tables. Every row is a question for a human.
4. **Answer them yourself.** Then write the Gherkin.

Try it on `specs/REQ-14.md` first — three sentences, and the perspectives will
find well over a dozen defects, none of which is a coding defect.

## The boundary this repo is built to protect

> **Agents find ambiguity. Humans resolve it.**

Finding scales. Resolving doesn't. Every prompt here ends with the same hard
rules — never resolve an ambiguity, never invent a value, never fill in the
Decision column — because the moment the agent answers its own questions you're
back to the pipeline grading its own homework, and your confidence goes up while
your correctness doesn't.

**Delegable** — checks internal to the document: ambiguity, testability,
contradiction, missing cases, terminology drift, traceability.

**Not delegable** — anything needing knowledge the model doesn't have: what the
business actually wants, what the regulator requires, what you tried in 2023
that opened a fraud hole, which of two stakeholders wins.

An agent can tell you "10% of what?" is undefined. Only you can decide the
answer, and deciding *is* the engineering.

## Layout

```
perspectives/tester.md                   write the test cases
perspectives/designer.md                 write the high-level design
perspectives/user.md                     write the help-centre article
perspectives/EXTRA-perspectives.md       eight more, incl. security, SRE, fraud, i18n
perspectives/_TEMPLATE.md                write your own

commands/run.md                          /apbr:run <spec-file-or-dir> [output-dir]
agents/pbr-tester.md                     blind subagent runner for tester.md
agents/pbr-designer.md                   blind subagent runner for designer.md
agents/pbr-user.md                       blind subagent runner for user.md
.claude-plugin/                          plugin + marketplace manifests

specs/REQ-14.md                          fixture requirement — deliberately not fixed
findings/REQ-14-expected-findings.md     reference baseline for regression-checking a run
runs/REQ-14/                             a committed reference run
docs/design-notes.md                     how blindness and the plugin root work
```

## Install it

This repository is a Claude Code plugin *and* its own marketplace:

```
/plugin marketplace add dariomac/agentic-pbr
/plugin install apbr@agentic-pbr
```

Then, in any project:

```
/apbr:run specs/REQ-14.md
/apbr:run docs/specs/ ./pbr-runs
```

The first argument is a spec file or a directory to walk; the second is where
runs are written, defaulting to `./pbr-runs`. Nothing is copied into your
project — the perspectives stay in the plugin, so updating the plugin updates
the technique.

Each subagent reads its own `perspectives/*.md` from the plugin at run time, so
those files stay the single source of truth. Blindness is enforced three ways:
separate contexts, an absolute two-files-only read rule, and an orchestrator
prompt that carries a spec path and nothing else. See `docs/design-notes.md`.

The orchestrator is bound by the same boundary as the readers: it consolidates
and counts, and it does not fill in the Decision column.

## Where it goes in the loop

A PBR gate sits **in front of** the outer BDD loop. A blocked row is a **red at
the spec level** — Red-Green-Refactor applied to the spec, before a line of
Gherkin exists.

And it's measurable, which matters: IEEE 610.12-1990 defined software
engineering as a *systematic, disciplined, **quantifiable*** approach. Blocked
rows per requirement, per perspective; ambiguity density; share of Gherkin
traceable to a specific spec clause.

## Where this is going

Open work, roughly in priority order:

- **Sharpen the three core perspectives.** Compare runs against
  `findings/REQ-14-expected-findings.md` after every edit; a perspective change
  that loses rows is a regression, not a refactor.
- **Promote perspectives out of `EXTRA-perspectives.md`.** Security, fraud &
  abuse, and i18n are the strongest candidates — each needs a full
  `perspectives/*.md` and a matching blind runner.
- **Per-perspective commands.** `/apbr:tester`, `/apbr:designer`, `/apbr:user`
  for running a single reader against a spec.
- **Build the internal tooling.** Metric extraction from a run directory,
  baseline diffing between runs, and a regression check that can go in CI.
- **Widen the spec corpus.** REQ-14 is one fixture and a friendly one. Real
  specs, longer specs, and specs that are genuinely clean — a technique that
  never reports "no blocked rows" is just a defect generator.
- **Test the approach with other professionals.** Have testers, designers, and
  technical writers run their own perspective by hand against the same spec,
  then compare. Where the agent and the human diverge is where the perspective
  file is wrong.
- **Measure divergence across models.** The same perspective on a different
  model should find substantially the same rows. If it doesn't, the perspective
  is underspecified.

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
