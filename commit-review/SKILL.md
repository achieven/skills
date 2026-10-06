---
name: commit-review
description: Use when the user asks to review a specific already-made commit (e.g. "/commit-review", "/commit-review abc123", "review commit abc123", "code review the last commit") OR the not-yet-committed changes in the working tree (e.g. "/commit-review uncommitted", "commit review my local changes", "review what I'm about to commit"). Produces a structured bullet-point report — what each changed section of code does, architectural/paradigm changes, security concerns, agent/skill changes, and package version bumps — and syncs docs/architecture.md via the architecture-diagram skill if the change touched the architecture. It is a descriptive report, not a bug hunt with fixes: distinct from /code-review (finds correctness bugs, can apply fixes) and /review (reviews a GitHub PR).
---

# Commit review

Reviews one specific commit — or the current uncommitted changes — end-to-end and reports
back in the fixed structure below.
This is a reporting skill: it reads and explains, it does not fix issues itself unless
the user separately asks.

## Steps

1. **Pick the mode and resolve the target.**
   - **Working-tree mode** — if the user says `uncommitted`, `working tree`, `staged`,
     `unstaged`, `local changes`, `pending`, `wip`, or "before I commit" (e.g.
     `/commit-review uncommitted`), review the not-yet-committed changes against `HEAD`.
     There is no commit to resolve; just confirm `HEAD` exists
     (`git cat-file -e HEAD^{commit}`) and skip to step 2. If `git status --short` is
     empty, tell the user there is nothing uncommitted and stop.
   - **Commit mode** — otherwise use the ref the user gave (SHA, branch, `HEAD`, `HEAD~2`,
     etc.); default to `HEAD` if none given. Verify it exists with
     `git cat-file -e <ref>^{commit}`; if that fails, tell the user and stop.
2. **Gather the diff.**
   - **Commit mode:**
     - `git log -1 --format='%H%n%an <%ae>%n%ad%n%s%n%b' <ref>` for identity/message.
     - `git show --stat <ref>` for the file list touched.
     - `git show <ref>` for the full patch — this is what you read to write the review.
     For a merge commit, use `git show --first-parent <ref>` so the diff isn't the whole
     branch.
   - **Working-tree mode:**
     - `git branch --show-current` and `git rev-parse --short HEAD` for context (no
       author/message exists yet).
     - `git status --short` to see staged, unstaged and untracked files; note which is which.
     - `git diff HEAD --stat` and `git diff HEAD` for tracked changes (staged + unstaged).
     - `git ls-files --others --exclude-standard` for untracked files, which `git diff`
       does not show. Read each with the Read tool; for large or data-like files (JSON,
       lockfiles, binaries) just report name and size and sample the head instead of
       dumping them. Flag untracked files that look like scratch/test data or secrets.
     - Label the review "Uncommitted changes on `<branch>` (based on `<sha>`)".
3. **Summarize what the code does**, in bullet points, one bullet per changed
   function/paragraph/logical section (not per line). State what it does, not a
   restatement of the diff syntax. Group bullets by file.
4. **Architectural / paradigm changes.** Call out things like: a new service or module
   boundary, a new communication style (e.g. added a message broker, switched sync→async),
   a new design pattern (DI, repository, event sourcing), restructured layering, a schema
   change, or a shift in how existing components talk to each other. If nothing rises above
   normal feature/bugfix work, say so explicitly rather than omitting the section.
5. **Security concerns.** Scan the diff for: hardcoded secrets/credentials/tokens, newly
   committed `.env`/key files, new exposed ports or loosened network exposure (e.g.
   `0.0.0.0`, new `ports:` in docker-compose), disabled/weakened auth or TLS, injection risk
   (string-built SQL/shell commands, unsanitized input into a query or command), overly
   permissive CORS, dependency bumps to versions with known CVEs (only if you actually know
   of one — don't speculate), and secrets passed via build args or plaintext env vars that
   end up in image layers. If none found, say so explicitly — don't skip the section.
6. **Agents/skills changes.** Check whether the diff touches `.claude/agents/**`,
   `.claude/skills/**`, `~`-scoped equivalents referenced in the repo, or `CLAUDE.md`. If so,
   explain what was added/changed/removed and what it's for. If not touched, say "none."
7. **Package version changes.** Diff dependency manifests touched by this commit (or, in
   working-tree mode, by `git diff HEAD`)
   (`package.json`/`package-lock.json`, `requirements.txt`/`pyproject.toml`/`uv.lock`,
   `go.mod`, etc.) and list each changed package as `name: old → new`. If none touched, say
   "none."
8. **Architecture diagram sync.** If step 4 found a change that would actually be visible
   in `docs/architecture.md` (new/removed service, changed ports/topology, new inter-service
   call, new datastore or pipeline stage) — invoke the `architecture-diagram` skill for this
   commit (pass it the resolved ref from step 1; in working-tree mode tell it to diff the
   working tree against `HEAD`). Report what it changed. If step 4 found no such change,
   state "no diagram update needed" and do not invoke the other skill.
9. **Present the review** to the user under these headers, in this order: `Summary`,
   `Architecture / paradigm changes`, `Security`, `Agents & skills`, `Package versions`,
   `Architecture diagram`. Keep each bullet to one line where possible.

## Notes

- Read enough surrounding context (`git show` with default context, or open the full file
  via the Read tool) to describe *why* a section exists when the diff alone is ambiguous —
  don't guess at intent from a bare `+`/`-` line.
- If the commit is large (many unrelated files), it's fine to group trivial/mechanical
  changes (formatting, lockfile churn) into a single bullet instead of one per file.
- This skill never runs `git commit`, `git push`, or any other state-changing git command —
  read-only.
