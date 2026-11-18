---
name: memory-audit
description: Use when the user asks to check, clean up or review Claude's saved memory ("/memory-audit", "audit my memories", "is my memory stale"). Compares MEMORY.md with the memory files for the current project, finds orphans, duplicates and stale references, and proposes fixes.
---

# Memory audit

The memory dir for this project is `~/.claude/projects/<project-path-with-dashes>/memory/` (see the path in the system prompt).

1. List the dir and read `MEMORY.md` and every memory file.
2. Check and report:
   - index lines pointing to missing files; memory files not in the index
   - missing/invalid frontmatter (`name`, `description`, `metadata.type`)
   - near-duplicate memories that should be merged
   - memories naming files, flags or functions: verify each still exists in the repo (grep) and flag those that do not
   - project memories with past dates or "later" plans that look done (check `git log`)
3. Output a short list: **OK / fix / delete** per memory, with a one-line reason.
4. Do not modify anything until the user approves the proposed changes.
