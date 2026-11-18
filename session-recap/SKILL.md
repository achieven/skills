---
name: session-recap
description: Use when the user asks for a recap of the current session ("/session-recap", "what did we do", "summarize this session", "where were we"). Produces a 5-line status - done, in progress, decisions, open questions, next step - from the conversation and git state.
---

# Session recap

1. Run `git status --short` and `git diff --stat` (and `git log --oneline -5` if commits were made this session).
2. From the conversation plus git state, output exactly:
   - **Done:** what was finished (files/areas touched)
   - **In progress:** anything half-done or uncommitted
   - **Decisions:** choices the user made, one line each
   - **Open questions:** anything awaiting the user's answer
   - **Next step:** the single most sensible next action
3. No more than ~12 lines total. Do not change anything.
