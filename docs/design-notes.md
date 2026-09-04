# Design notes — how the plugin runs a PBR pass

Everything needed to reproduce a Perspective-Based Reading pass ships inside the
`apbr` plugin, so installing it is enough — no files are copied into the
consuming project.

```
.claude-plugin/plugin.json        plugin manifest (name: apbr)
.claude-plugin/marketplace.json   this repo doubles as the marketplace
commands/run.md                   /apbr:run <spec-file-or-dir> [output-dir]
agents/pbr-tester.md              blind runner for perspectives/tester.md
agents/pbr-designer.md            blind runner for perspectives/designer.md
agents/pbr-user.md                blind runner for perspectives/user.md
perspectives/                     the single source of truth for each role
```

## Why the agents hold no perspective content

Each agent reads `${CLAUDE_PLUGIN_ROOT}/perspectives/<role>.md` at run time and
treats it as authoritative. Edit the perspective file and the agent's behaviour
changes with it — there is exactly one copy of each perspective, and it ships
with the plugin rather than being vendored into every project that uses it.

`${CLAUDE_PLUGIN_ROOT}` is substituted before the agent's instructions reach it,
so the agent is handed a real absolute path and never sees a placeholder. This
was verified empirically, not assumed.

## Blindness is enforced three ways

One way is not enough:

1. *Separate contexts.* Each perspective is a subagent, so it cannot see the
   others' reasoning or output.
2. *An absolute read rule.* Each agent may open exactly two files: its own
   perspective, and the spec it was handed. The rule is stated as a principle
   rather than a list of forbidden paths, because the plugin runs in
   repositories whose layout it has never seen — and in a real codebase the
   answers are usually lying around in a test, an ADR, or a sibling spec.
   Filling a gap from context is the exact failure the technique exists to
   prevent.
3. *A clean prompt.* The orchestrator passes a spec path and an output path.
   No spec text, no hints, no example findings. See `commands/run.md`.

**Tools are narrowed to `Read, Write`** — enough to read two files and write one,
not enough to go exploring. `Glob` is deliberately absent: in someone else's
repository it is a licence to wander.

**Spec text is data, not instruction.** Each agent is told to treat the spec's
contents as the artifact under inspection. A spec is a document under review; if
one contains a line addressed to the reader, that is a finding, not an order.

**Writes are confined to the output directory.** The readers never edit a spec,
and the orchestrator never writes outside the path it was given.

## Where the rest lives

Adding a perspective, the sideload development loop, and the regression
discipline against `findings/` are in [CONTRIBUTING.md](../CONTRIBUTING.md).
This file covers only *why* the machinery is built the way it is.
