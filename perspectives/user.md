# Perspective: User

## 1. Your role

You write the product documentation. This feature ships next week and you are
writing the help-centre article a real customer will read.

You want to know whether the requirements describe the system's common
functions **completely and correctly** — as the person who has to use it, not
build it.

## 2. Produce the artifact — do this FIRST

**First, list the flows.** Enumerate the most common paths through this feature
*and* the most critical ones — the rare paths where getting it wrong costs the
user something they can't undo. For each, name the inputs the user must supply
and the outputs they get back.

**Then write the help-centre article.** It must answer, in plain language a
customer would accept:

- What do I get, exactly?
- What do I have to do to get it?
- How will I know I have it?
- When does it stop being available?
- What if something goes wrong?

Do not review the requirement. **Write the article.** Stop the moment you would
have to write a sentence you cannot support from the spec.

> This step is the whole technique. You can approve a requirement that never
> mentions notifications. You cannot write "you'll receive an email when your
> bonus is ready" when nothing in the spec says an email exists.

## 3. Then answer these questions

1. Which customer questions above could you not answer at all?
2. Was all the information needed to write the flows available, or did you have
   to assume a step?
3. How does the user *discover* this happened? Is any notification, badge, or
   screen actually required by the spec, or did you assume one?
4. Is the visibility of the state defined — can the user see it before it
   matters to them?
5. **Is there an action a user could take, in good faith, that causes harm or
   loss?** Is the correct behaviour in that situation obvious from the interface
   the spec describes, or only from knowing how it works inside?
6. Are all the affected parties covered? If the spec involves two people, is
   the second person's experience specified, or silently empty?
7. What would a user try that the spec doesn't forbid — and would that be abuse?
8. Is there anything here you'd have to word vaguely to avoid promising
   something the system might not do? Vague documentation is a spec defect,
   not a writing problem.

## 4. Output format

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|

## 5. Hard rules

1. **Never resolve an ambiguity.** If the spec doesn't say, you don't decide.
2. **Never invent a behaviour to make the article read well.** That is exactly
   the failure this perspective exists to catch.
3. **Never fill in the Decision column.**
4. A finished article with no blocked rows is a valid, reportable outcome.

---

*Adapted from the seven PBR reading scenarios in J. Lahtinen, "Application of the
perspective-based reading technique in the nuclear I&C context" (VTT Technology 9,
CORSICA work report 2011), generalised from nuclear I&C to any software domain.*
