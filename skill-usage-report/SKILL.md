---
name: skill-usage-report
description: Use when the user asks which skills they use most/least, wants a skill usage report, or asks which skills are unused and could be pruned ("/skill-usage-report", "which skills do I use", "skill reviewer"). Counts Skill tool calls and /slash invocations across all session transcripts and compares with installed skills.
---

# Skill usage report

Read-only. Counts how often each skill was invoked, across all projects, and flags unused ones.

## Steps

1. Count Skill-tool invocations (model-triggered or slash-triggered) from transcripts:
   ```
   grep -ho '"name": *"Skill", *"input": *{ *"skill": *"[^"]*"' ~/.claude/projects/*/*.jsonl ~/.claude/projects/*/*/subagents/*.jsonl 2>/dev/null \
     | sed 's/.*"skill": *"//;s/"$//' | sort | uniq -c | sort -rn
   ```
2. Count user-typed slash commands (`<command-name>/x</command-name>`):
   ```
   grep -ho '<command-name>/\?[a-zA-Z0-9:_-]*</command-name>' ~/.claude/projects/*/*.jsonl \
     | sed 's/<[^>]*>//g;s#^/##' | sort | uniq -c | sort -rn
   ```
   Ignore built-ins (clear, help, model, config, ...); keep only names that are skills.
3. List installed skills: directories with `SKILL.md` under `~/.claude/skills/` and the current project's `.claude/skills/`.
4. Report a table: skill | scope (global/project) | uses | last used (file mtime of the newest transcript that mentions it; optional).
5. Call out: top 3, installed-but-never-used skills (candidates to delete or to improve the `description` so they trigger), and used-but-not-installed names (plugin/built-in).

Keep it short. Do not edit or delete skills; only suggest.
