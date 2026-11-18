---
name: todo-sweep
description: Use when the user asks for TODOs/FIXMEs in the code, or wants leftover work found before finishing ("/todo-sweep", "any TODOs left", "find FIXMEs"). Lists TODO/FIXME/HACK/XXX markers, grouped by file, and flags ones added in the current uncommitted diff.
---

# TODO sweep

Read-only.

1. Find markers, excluding vendor/build dirs:
   ```
   grep -rnE 'TODO|FIXME|HACK|XXX' --exclude-dir={node_modules,.git,dist,build,.venv,coverage} . | head -200
   ```
2. Find the ones newly added by uncommitted work:
   ```
   git diff HEAD -U0 | grep -E '^\+.*(TODO|FIXME|HACK|XXX)'
   ```
3. Report: count per marker type, then a list grouped by file (`path:line - text`), with **new in this change** listed first.
4. Say which look stale or risky (e.g. FIXME in a hot path). Do not fix anything unless asked.
