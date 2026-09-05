# Extra perspectives — the ones nobody could staff in 1995

> **Note.** Four perspectives that once lived here as sketches are now shipped in
> full: `maintainer.md`, `verifier.md`, `regulator.md` and `contractor.md`. What
> remains below is still unbuilt. The ones marked *(overlaps …)* are close to a
> shipped perspective — check it first, and only build a new one if it finds
> something the existing reader genuinely cannot.

PBR shipped with three perspectives because three trained readers per
requirement was already more than most projects could afford. That constraint
is gone. Each of these follows the same shape: **produce an artifact, then
report what you couldn't produce.** Copy `_TEMPLATE.md` and fill in the middle.

| Perspective | Produce this artifact | Catches |
|---|---|---|
| **Security** | The threat model — actors, assets, trust boundaries, abuse cases | Missing authz, unbounded values, replay, enumeration |
| **Operability / SRE** | The runbook and the alerts for when this breaks at 3am | No observability, no failure semantics, silent partial states *(overlaps `verifier`)* |
| **Data & privacy** | The data inventory — what's collected, retained how long, on what legal basis | Undeclared PII, no retention rule, no deletion path |
| **Fraud & abuse** | The attack plan: how would you exploit this for profit? | Self-referral, farming, stacking, timing attacks |
| **Accessibility** | The keyboard-and-screen-reader walkthrough | Interactions that assume sight or a mouse |
| **i18n / localisation** | The same flow in another locale and currency | Currency, timezone, name and address assumptions, RTL |
| **Cost to serve** | The unit-economics estimate per invocation | Unbounded discounts, unbounded fan-out, unbounded retries |
| **Legal / regulatory** | The disclosure text this feature requires | Promotional-terms rules, consumer-protection duties *(overlaps `regulator`, which reads the document against its obligations rather than drafting disclosure copy)* |

In 1995 each of these cost you a trained specialist and a meeting. That is the
entire reason inspections died. It isn't the reason any more.
