---
name: scaffold-project-docs
description: Use at the end of the planning step of a feature or project, once the user has APPROVED the plan, to write the repo's reference docs - docs/product-spec.md, docs/components.md, docs/architecture.md, docs/backlog.md (TODOs) and docs/tests.md - and wire them into CLAUDE.md so every future session reads them. Also use to bring these docs in line with an approved plan on an existing repo. Do not use before the plan is approved, and not for small code changes (those just update the affected doc sections).
---

# Scaffold project docs (from an approved plan)

The five docs are the repo's long-lived memory: every future session reads them before changing code, so they must be
short, accurate, and honest about what is built versus what is only intended.

## When

Run only **after the user approved the plan** for a feature/project. Source material is the approved plan and the planning
conversation, not guesses. Anything the plan does not settle is **not invented**: list it under "Open questions" in the
product spec and ask the user (global rule: ambiguity means stop and ask for the affected part).

## Steps

1. **Check what exists.** List `docs/` and read `CLAUDE.md`. If docs already exist, **update them in place** (merge the plan
   in, keep existing content and numbering, do not rewrite wholesale). Never change the intent sections of an existing
   product spec (see below) unless the plan explicitly changes the intent; say so when it does.
2. **Write the docs** (templates below). Keep each one scannable: headings, short bullets, tables. No fluff, no secrets
   (no keys, passwords, connection strings), no duplication between docs (link instead).
3. **Wire them in.** Create or update `CLAUDE.md` with a short "Read before changing code" list linking the five docs and
   these rules: update the docs in the same change as behaviour changes; check changes against the guardrail checklist;
   known gaps are not fixed as a side effect; run the `check-guardrails` skill before finishing a feature. Keep any
   existing CLAUDE.md content.
4. **Diagram.** Produce `docs/architecture.md` through the `sync-architecture-diagram` skill (do not hand-roll it).
5. **Report** the files created/changed, and the open questions you left for the user.

## Templates

### docs/product-spec.md
- `# Product spec and guardrails`, plus a legend: **INTENDED** (what the product should be) vs **CURRENT** (what is built).
- `## 1. The idea`: the problem, who it is for, what the user gets. (Intent.)
- `## 2. Core principles` and `## 3. Modes/variants`: the rules the product must follow, e.g. strictness or behaviour options. (Intent.)
- `## 4. End-to-end flow`: steps from request to result, and the **status lifecycle** (each status, who moves it, terminal states, failure state with `error`).
- `## 5. Data and storage rules`: what is stored where, what must never be stored/exposed, durability expectations.
- `## 6. Guardrails: checklist for judging any change`: a **numbered** list of yes/no rules a reviewer can check against a diff
  (e.g. "every unfulfilled request ends `failed` with an `error`", "no hardcoded tolerances", "no secrets in code"). Mark which
  ones are checked mechanically (script/CI) and which need judgement. The `check-guardrails` skill consumes this section.
- `## Known gaps (INTENDED vs CURRENT)`: gaps accepted for now; they are known and not fixed as a side effect.
- `## Open questions`: unresolved items from planning.

### docs/components.md
One section per deployable unit (service, consumer, job, datastore, external API), grouped (request path, workers, storage, external).
Each section: **owns** (responsibilities), **must not do** (boundaries), inputs/outputs (endpoints, topics, tables), port/image/tech,
how it fails (retry, DLQ, terminal status). Startup order if it matters. Link to the code directory, not code.

### docs/architecture.md
Created by `sync-architecture-diagram`: Mermaid topology, component table, data-flow list, "Last verified against" line.

### docs/backlog.md
Prioritised TODO list, not a changelog.
- `## Next` (ordered, top is next), `## Later`, `## Ideas / unconfirmed`, `## Done` (one line each, move items here, do not delete).
- Each item: a checkbox, one-line outcome, why it matters, and a pointer to the spec section / guardrail / test it relates to.
- Seed it from the plan: the phases not yet built, the "Known gaps", and follow-ups the plan deferred.

### docs/tests.md
- **Strategy**: test pyramid for this repo, which language/runner per component (tests in the language of the code under test),
  everything runs in containers, what a new stage/feature must ship with (happy path, poison pill, transient failure, idempotency).
- **How to run**: the exact commands (container-based), and what CI runs versus what runs only locally.
- **Coverage map**: table `area | test files | what is covered`, from the plan.
- **Tests backlog**: tests that are planned, or exist but are off in CI, each with why and what is needed to turn them on
  (table: `test | where | why off/missing | to enable`).

## Rules

- Mark every statement as intended or current when they differ; never describe unbuilt behaviour as present.
- Do not add anything the plan did not decide; ask instead.
- Keep docs consistent: a status, topic, or service name must be spelled the same everywhere; update all docs that mention it.
- After the first scaffold, later changes touch only the affected sections, in the same change as the code.
