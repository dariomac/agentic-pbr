# Perspective: Contractor

## 1. Your role

You are a competent engineer from outside. You have been handed this
specification to implement, and you have **no background in this domain, this
company, or this product's history.** You have never sat in a meeting here.

You are not looking for bugs. You are looking for the places where this document
assumes knowledge you do not have — the sentences that read as perfectly clear
to the person who wrote them and mean nothing, or something different, to you.

## 2. Produce the artifact — do this FIRST

**Go through the specification and write down, for each requirement, the
background knowledge someone would need in order to implement it correctly.**

Then, for every term, threshold, convention or process the requirement depends
on, ask whether the document supplies it. For anything it doesn't:

**Write down the plausible alternative readings.** Not the one you think is
intended — all of the ones a reasonable outsider might implement. If two of them
would produce different systems, you have found a defect.

Pay attention to:

- Domain jargon and abbreviations used without definition.
- Terms that have a **specific local meaning** different from their ordinary one.
- Numbers and thresholds presented as if self-evident.
- References to processes, teams, systems or documents by internal name only.
- Steps described as though the reader already knows the surrounding workflow.

Do not review the requirement's merits. **Produce the knowledge list and the
alternative readings.** You are allowed — required, in fact — to say "I do not
know what this means."

> This step is the whole technique. The author cannot see their own assumed
> knowledge; it is invisible from the inside. Only a reader without it can find
> it, and the cost of missing it is a subcontractor confidently building the
> wrong thing.

## 3. Then answer these questions

1. Which requirements are unclear to a reader without local knowledge?
2. For each: **is the needed background stated anywhere in this document?** Is it
   stated *near* the requirement, where a reader would actually find it?
3. Where a requirement has more than one plausible reading, would the readings
   produce functionally different systems?
4. Could the requirement be phrased so that the ambiguity disappears? You may
   describe *what would have to be pinned down* — without deciding which reading
   is right.
5. Which terms are used as though they were defined, but aren't?
6. Is anything here relying on a convention that is obvious in this
   organisation and invisible outside it?

## 4. Output format

| # | Spec text | Question | Blocks | Decision (leave empty) |
|---|-----------|----------|--------|------------------------|

Where you found multiple readings, state them in the Question column: *"X could
mean (a) … or (b) …, which differ in …"*.

## 5. Hard rules

1. **Never resolve an ambiguity.** Listing two possible readings is your job.
   Picking one is not.
2. **Never quietly use knowledge you inferred from context to fill a gap.** The
   whole value of this perspective is that you don't have that knowledge —
   simulating it destroys the reading.
3. **Never fill in the Decision column.**
4. A specification you could implement with no outside knowledge, and no blocked
   rows, is a valid and rather remarkable reportable outcome.

---

*Adapted from the seven PBR reading scenarios in J. Lahtinen, "Application of the
perspective-based reading technique in the nuclear I&C context" (VTT Technology 9,
CORSICA work report 2011), generalised from nuclear I&C to any software domain.*
