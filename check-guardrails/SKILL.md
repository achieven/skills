---
name: check-guardrails
description: Use before declaring a feature or change done, or when the user asks to "check the guardrails" / "review against the spec". Runs the repo's mechanical guardrail checks if it has any, then judges the diff against the guardrail checklist in the repo's product spec and reports pass / fail / n-a per item. Report-only.
---

# Check guardrails (generic)

Judge the current change (working tree + commits since the default branch) against the repo's guardrails. Report only; do not fix unless asked.

## Steps

1. **Find the rules.** Read the project `CLAUDE.md`, then `docs/product-spec.md` (look for a "guardrails"/"checklist" section and a
   "Known gaps" section), `docs/components.md`, and `docs/architecture.md`. If there is no guardrail checklist, say so and fall
   back to the user's global rules (containers only, no secrets in output, TypeScript, DB defaults, etc.) and the spec's stated intent.
2. **Mechanical part.** If the repo has a guardrail runner (e.g. `scripts/guardrails/run.sh`, a Makefile target, a CI workflow named
   for guardrails/lint), run it the way the repo says (usually inside a container; do not start services on the host). List each
   finding verbatim. If it cannot run (Docker down), say so and review those items by hand.
3. **Read the diff.** `git diff $(git merge-base HEAD <default-branch>)` plus untracked files from `git status --short`.
4. **Judgement part.** For each checklist item the mechanical check does not decide, answer from the diff with `file:line`
   evidence. Read the mechanical result critically: grep-level checks miss indirect violations. Also check the universal items:
   failures end in a terminal failed state with an error (never silently stuck), stages have a DLQ, no secrets committed,
   no new tolerances/limits hardcoded when the spec says they are configurable.
5. **Known gaps.** Mention a listed gap only if the change makes it worse; never propose fixing it as a side effect.
6. **Docs sync.** If behaviour described in the spec or components doc changed, name the sections to update; topology changes
   need the **sync-architecture-diagram** skill.

## Report format

A table `# | guardrail | pass / fail / n-a | evidence (file:line)`, then a short "Needs your decision" list for anything that
conflicts with the spec. Use `n-a` when the change does not touch an item; do not pad.
