---
name: draft-commit-message
description: Use this skill when the user explicitly signals that the current task/change is finished (e.g. "that's it", "done", "looks good, that's all", "ship it") after code changes were made earlier in the session. Drafts a git commit message summarizing the session's changes and writes it into .git/COMMIT_EDITMSG so it shows up pre-filled in the IDE's source control commit box — it does NOT run `git add` or `git commit`.
---

# Draft commit message (no commit)

Trigger: the user says the task is done, right after you (Claude) made or guided code changes in this conversation. Do not trigger on every message — only on an explicit completion signal.

## Steps

1. Find the repo root for the relevant changes: `git rev-parse --show-toplevel`. If the user has been working across multiple repos in the session, use the one whose files were actually just edited.
2. Run `git status --short` and `git diff` (staged + unstaged) to see exactly what changed on disk right now — the message must reflect the real diff, not just your memory of the conversation.
3. Check recent commit style with `git log -5 --format="%s%n%b---"` and roughly match it (this repo has no strict convention observed so far — short imperative subject, optional body).
4. Draft a concise commit message:
   - Subject: imperative mood, ≤ 70 chars, summarizes the "why" not a file-by-file list.
   - Body (optional, only if there are multiple distinct changes worth calling out): short bullet points, one line each, matching how you already summarized the work to the user earlier in conversation — don't re-derive from scratch if you already wrote a clean summary.
   - No `Co-Authored-By` trailer — this is a draft the user will review/edit before committing themselves, not a real commit.
5. Write the message as plain text to `<repo-root>/.git/COMMIT_EDITMSG`, ending with a trailing newline. Overwriting this file is safe — it's just the default message pre-filled into the next commit dialog, not commit history.
6. Do NOT run `git add`, `git commit`, `git stash`, or any other state-changing git command. This is a preview only.
7. Tell the user in one short sentence that the draft message is ready in the IDE's source control panel for review before they commit.
