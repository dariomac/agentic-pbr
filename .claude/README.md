# .claude — running this repo's PBR pass with agents

Everything needed to reproduce a Perspective-Based Reading pass lives here, in
the repository, so a checkout is enough.

```
.claude/agents/pbr-tester.md      blind runner for perspectives/tester.md
.claude/agents/pbr-designer.md    blind runner for perspectives/designer.md
.claude/agents/pbr-user.md        blind runner for perspectives/user.md
.claude/commands/pbr.md           /pbr <spec> — orchestrates all three
runs/<SPEC-ID>/                   output: one file per perspective + consolidated
```

Run it:

```
/pbr specs/REQ-14.md
```

## Design notes

**The agents hold no perspective content.** Each one reads
`perspectives/<role>.md` at run time and treats it as authoritative. Edit the
perspective file and the agent's behaviour changes with it — there is exactly
one copy of each perspective in the repo.

**Blindness is enforced three ways**, because one way is not enough:

1. *Separate contexts.* Each perspective is a subagent, so it cannot see the
   others' reasoning or output.
2. *An explicit deny-list.* Each agent is told, by name, which files it may not
   open: the other perspectives, `findings/` (the reference baseline),
   `README.md`, and any sibling's output file.
3. *A clean prompt.* The orchestrator passes a spec path and an output path.
   No spec text, no hints, no example findings. See `commands/pbr.md`.

**Tools are narrowed to `Read, Write, Glob`** — enough to read two files and
write one, not enough to go exploring.

**Spec text is data, not instruction.** Each agent is told to treat the spec's
contents as the artifact under inspection. A spec is a document under review; if
one contains a line addressed to the reader, that is a finding, not an order.

## Adding a perspective

Copy `perspectives/_TEMPLATE.md` to `perspectives/<name>.md` (or lift one from
`perspectives/EXTRA-perspectives.md`), then copy any `.claude/agents/pbr-*.md`
to `pbr-<name>.md` and change four things: the frontmatter `name` and
`description`, the perspective path it reads, the deny-list (add the perspective
you just cloned from, remove your own), and the row-id prefix.
