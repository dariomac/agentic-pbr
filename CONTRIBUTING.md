# Contributing

This repository is the working home of the technique, not a frozen artifact.
The perspectives are meant to get sharper, and the fastest way to sharpen them
is to run them against real specs and compare.

If you only want to *use* `apbr`, the [README](README.md) is the right door.

## Repository layout

This repo is both the `apbr` plugin and the marketplace that serves it.

```
perspectives/tester.md                   write the test cases
perspectives/designer.md                 write the high-level design
perspectives/user.md                     write the help-centre article
perspectives/maintainer.md               map functions, dependencies, rationale   🧪
perspectives/verifier.md                 negative requirements, failure analysis 🧪
perspectives/regulator.md                obligations, references, consistency    🧪
perspectives/contractor.md               assumed knowledge, alternative readings 🧪
perspectives/EXTRA-perspectives.md       further sketches, incl. security, fraud, i18n
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

The perspective files are the single source of truth. The agents hold no
perspective content — each reads `${CLAUDE_PLUGIN_ROOT}/perspectives/<role>.md`
at run time, so editing a perspective changes the agent's behaviour with it.

## The development loop

The plugin is this repository, so test a change by sideloading the checkout —
no publishing, no install:

```bash
claude --plugin-dir /path/to/agentic-pbr
```

Then run `/apbr:run specs/REQ-14.md` from a scratch directory. Because the
agents resolve their perspectives through `${CLAUDE_PLUGIN_ROOT}`, a sideloaded
checkout behaves exactly like an installed plugin.

## Changing a perspective is a change to the instrument

`findings/REQ-14-expected-findings.md` is the reference baseline: 21 rows a
healthy run reproduces. Run the pass before and after your edit and compare.

**A perspective change that loses rows is a regression, not a refactor.** If
your edit drops rows the baseline expects, either the edit is wrong or the
baseline is — say which, in the PR, and be specific about why.

Gaining rows is usually good, but check them. A perspective that produces more
rows by asking vaguer questions has got worse, not better. The test is whether a
human could answer each row in one sentence.

`runs/REQ-14/` holds a committed reference run for comparison. Note that runs
are not deterministic: two passes over the same spec with the same perspective
will differ at the margins. Look for rows that disappear systematically, not for
an exact diff.

## Adding a perspective

1. Copy `perspectives/_TEMPLATE.md` to `perspectives/<name>.md`, or lift one of
   the sketches from `perspectives/EXTRA-perspectives.md`.
2. Copy any `agents/pbr-*.md` to `agents/pbr-<name>.md` and change four things:
   the frontmatter `name` and `description`, the perspective path it reads, the
   row-id prefix, and the output skeleton.
3. Run it against `specs/REQ-14.md` and against a spec of your own.

The bar for a new perspective is that it finds something the existing three
don't. If its rows are a subset of the tester's, it isn't a perspective — it's
the tester with a different hat, and it will cost a subagent per run forever.

To expose a perspective as its own command later, add `commands/<name>.md` that
spawns the single agent. The namespace is already shaped for it: the plugin is
`apbr`, so every command reads `/apbr:<verb>`.

## Choosing a set of perspectives — an open hypothesis

**Nothing in this section is validated.** No profile below has been tested
against anything. They are recorded here, rather than in the README, because the
README is where users are told what to rely on and this is not yet that. Treat
them as hypotheses to attack.

### The move worth learning

The seven perspectives shipped here are not universal. They are one domain's
selection, generalised. The source report derives them in its §4.2 by asking
what is true about *its* situation — systems that outlive their authors,
work subcontracted down a chain, a regulator in the loop, safety consequence —
and the seven fall out of those facts. Maintainer exists because of the first,
contractor because of the second, regulator and verifier because of the last two.

So the transferable thing is the derivation, not the list. For a spec of your
own, ask:

1. **Who is harmed if this is wrong, and how badly?** Consequence selects for
   `verifier`; reversible, low-stakes work does not need it.
2. **Who implements it — someone in this conversation, or someone outside it?**
   Distance selects for `contractor`.
3. **How long will it live, and who changes it next?** Longevity selects for
   `maintainer`.
4. **Does anyone have to be shown that the rules were followed?** Auditability
   selects for `regulator`.
5. **Does a person outside the company live with the result?** If not, `user`
   is weaker than it looks and an internal-operator reading matters more.

Then pick the readers those answers point at. Keep `tester` and `designer` in
almost every set — they are the fundamental two.

### Illustrations, not recommendations

Each of these is a guess at what the questions above would produce. The second
column matters as much as the first: **a set is defined by what it leaves out**,
and the omission is what tells you when the set is wrong for you.

| Situation | Readers | Deliberately does not read for |
|---|---|---|
| Customer-facing feature | tester, designer, user | Longevity, compliance evidence, outsider comprehension |
| Back-office / internal tool | tester, designer, maintainer | Customer comprehension — there is no external customer |
| API or integration | designer, tester, contractor | End-user experience; the consumer is another system |
| Regulated or auditable | regulator, verifier, tester | Usability and maintainability |
| Outsourced build | contractor, designer, regulator | Day-to-day usability; the risk is misinterpretation |
| Change to existing system | maintainer, verifier | Anything about the new behaviour in isolation |

### Why this is not shipped as a feature

There is no `@profile` shorthand and the default remains `tester,designer,user`.
A named shorthand reads as a validated feature, and a set is a **stronger** claim
than a perspective: it asserts that these readers are sufficient together and
that the omitted ones are not needed. Four of the seven are still 🧪 with no
baseline, so a validated claim about combinations is not available yet.

The failure mode being avoided is a wrong set propagating its blind spot,
silently, to everyone who adopts it — and sets are exactly the kind of thing
nobody revisits once they are named.

### What would move a profile into the README

The same bar as a perspective, one level up: runs across several specs showing
the set has decent coverage and low internal redundancy — no member whose rows
are consistently contained in another member's.

Note that subsumption is an argument about *sets*, not about deletion. If reader
A's findings sit inside reader B's, that only means the two are wasteful in the
same pass. A may still be worth running alone as a cheaper, narrower probe, or
alongside a third reader without B. Measure it to compose sets, never to cull the
catalogue.

### The hypothesis most worth attacking

That profiles help at all. Running the default three and adding one reader when
something looks off may simply beat choosing a set up front. If that turns out to
be true, this section should be deleted rather than expanded — and finding that
out is a more useful contribution than adding a seventh row to the table.

## Rules that are not up for negotiation

These are the technique, not style preferences. A change that weakens one of
them needs to argue the case explicitly.

1. **Readers stay blind.** Separate contexts, two files only, no hints in the
   prompt. Readers who see each other converge, and a converged pass finds what
   one reader would have found alone.
2. **No agent fills in the Decision column.** Not the readers, not the
   orchestrator, not "obviously it's the subtotal" dressed as a default.
3. **No agent invents a value.** A plausible invention is worse than a blocked
   row, because it disappears into the code.
4. **Readers open exactly two files.** Their perspective and the spec. In a real
   codebase the answers are usually lying around in a test or an ADR, and
   reading them resolves the ambiguity silently — the exact failure the
   technique exists to prevent.

`docs/design-notes.md` explains how each of these is enforced.

## What needs doing

Roughly in priority order:

- **Sharpen the three core perspectives**, against the baseline as described above.
- **Promote further perspectives out of `EXTRA-perspectives.md`.** Security and
  fraud & abuse are the strongest remaining candidates — each needs a full
  `perspectives/*.md` and a matching blind runner. Check first whether the
  shipped `verifier` already covers it.
- **Graduate the four experimental perspectives.** Maintainer, verifier,
  regulator and contractor ship marked 🧪: adapted from published scenarios, but
  with no baseline of their own and no evidence yet about which earn their cost
  or how much they overlap the stable three. The source report says plainly that
  only real use reveals which scenarios deserve to stay. To move one out of
  experimental it needs a reference baseline of its own under `findings/`, and
  runs against specs nobody wrote for it. Reports of a perspective performing
  *badly* are as useful as reports of it working.
- **Per-perspective commands.** `/apbr:tester`, `/apbr:designer`, `/apbr:user`
  for running a single reader against a spec.
- **Internal tooling.** Metric extraction from a run directory, baseline diffing
  between runs, and a regression check that can go in CI.
- **Widen the spec corpus.** REQ-14 is one fixture and a friendly one. Real
  specs, longer specs, and specs that are genuinely clean — a technique that
  never reports "no blocked rows" is just a defect generator.
- **Test the approach with other professionals.** Have testers, designers, and
  technical writers run their own perspective by hand against the same spec,
  then compare. Where the agent and the human diverge is where the perspective
  file is wrong. This is the most valuable contribution anyone can make, and it
  needs no code.
- **Test whether choosing a set of perspectives helps at all.** See the open
  hypothesis above. The comparison is a chosen set against "run the default
  three, add a reader when something looks off". A negative result deletes that
  section, which is a perfectly good outcome.
- **Measure per-perspective unique yield.** Across several specs, what fraction
  of a reader's rows did only that reader produce? Counting discovered rows
  only — pre-announced gaps inflate both corroboration and apparent redundancy.
  This is the evidence that lets a perspective lose its 🧪, and the same numbers
  compose sets.
- **Measure divergence across models.** The same perspective on a different
  model should find substantially the same rows. If it doesn't, the perspective
  is underspecified.

## Reporting a run

If you run `apbr` against a real spec and something interesting happens — a
perspective missed an obvious gap, or invented a value, or produced rows nobody
could answer — that is worth an issue. Include the spec (redacted is fine), the
run output, and which model you used. Failure reports on real requirements are
more useful than opinions about the prompts.
