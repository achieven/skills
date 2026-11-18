---
name: standup-summary
description: Use when the user wants a quick standup/"what did I do" summary of recent work ("/standup-summary", "what did I do yesterday", "daily summary"). Builds it from the user's git commits in the current repo over the last day (or a period they name).
---

# Standup summary

1. Period: default "yesterday and today" (`--since="1 day ago"`); use the user's period if given. Use `git config user.name` as author.
2. Run `git log --all --author="$(git config user.name)" --since="<period>" --pretty='%h %ad %s' --date=short`.
3. Group commits by theme (feature/fix/docs/chore), not chronologically, and write 3-6 plain bullets in past tense ("Added X", "Fixed Y").
4. Add one line "Next:" only if the conversation or branch names make it obvious; otherwise omit it.
5. Output is paste-ready text. Read-only.
