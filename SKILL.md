---
name: memo
description: "Maintain a tiny MEMORY.md at the project root that survives context compaction — capturing goal, core context, progress, decisions, pitfalls. Activated by /memo. On first run, creates MEMORY.md; on subsequent runs, loads it and resumes. Then keeps it updated through the session as milestones, decisions, and pitfalls appear."
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep]
---

# memo

Persistent task-scoped working memory for one project.

## Activation

Invoked **only** via `/memo`. Two shapes:

- `/memo` — bootstrap or resume (no payload).
- `/memo <prompt>` — bootstrap or resume, then handle the user's prompt with MEMORY.md already loaded.

## File

A single `MEMORY.md` at the **project root** (`git rev-parse --show-toplevel`, fallback to CWD).

## Decision tree

On `/memo` invocation:

1. **If `MEMORY.md` exists** → resume:
   - Read it.
   - Reply with a one-line acknowledgement: the Goal line + `◀ next: <first unchecked progress item>`.
   - Do **not** echo the file.
   - Then handle the trailing user prompt (if any) using the loaded context.

2. **If `MEMORY.md` does not exist** → bootstrap:
   - Resolve the task's goal:
     - If the trailing prompt clearly states the goal, use it.
     - Otherwise ask: "What's the goal in one sentence? Any hard constraints? First step?"
   - Write `MEMORY.md` from the template below.
   - Reply: `📝 MEMORY.md created` + the Goal line.
   - Then handle the trailing prompt (if any).

## Template

Write exactly this structure. Cap total length at ~300 lines. Empty sections get `_(none yet)_`.

```markdown
# MEMORY

> Created: <YYYY-MM-DD> · Last updated: <YYYY-MM-DD HH:MM>

## Goal
<1–3 sentences. What "done" looks like. Concrete and verifiable.>

## Core Context
- <Key file/path — one-line role>
- <Architectural constraint>
- <External dep + version + where its config lives>
<Cap ~8 bullets. Each must save a future session a search.>

## Progress
- [ ] <step>  ← cursor
- [ ] <step>

## Decisions
- <Decision> — <one-line why>
<Cap ~5. Only ones a future session would otherwise re-litigate.>

## Pitfalls
- _(none yet)_
<Things that already cost time. Not generic advice.>

## Open Questions
- _(none yet)_
```

## Update triggers

While the session continues, update `MEMORY.md` **without asking** when any of these happen:

1. A progress item finishes → check it off, advance the cursor.
2. A non-trivial bug or footgun bites → add to Pitfalls (one line).
3. A real decision is made → add to Decisions with the *why*.
4. The user reveals a hard constraint → add to Core Context.
5. A key file is discovered → add to Core Context with its role.

Do **not** update for: routine reads, exploratory edits, every sub-step.

After updating, emit one short line: `📝 memo: <what changed>`. Never dump the file.

Use `Edit` for surgical changes. Use `Write` only when rewriting from scratch.

## Authoring rules

**Include**: file paths + their role, non-obvious constraints, bugs already hit + the fix, exact commands/env-vars annoying to rediscover, pointers like `src/x.ts:142`.

**Exclude**: project-wide conventions (those belong in CLAUDE.md/AGENTS.md), code blocks > 5 lines (link instead), generic best practices, obvious facts, conversational filler, raw stack traces.

**Hard rule**: if a line wouldn't save a future session more tokens than it costs, delete it.

## Size discipline

If `MEMORY.md` exceeds ~400 lines, prune before adding anything new:
- Scratch entries older than 24h → promote or delete.
- Pitfalls no longer applicable → delete.
- Decisions now obvious from the code → delete.
- Finished progress sub-bullets → collapse to one line.

If the user asks to dump something large into the memo, refuse and link to a separate file instead.

## Resources

- `assets/MEMORY.template.md` — empty template, ready to copy.
- `references/authoring-guide.md` — read when unsure what belongs in the file.
- `references/examples.md` — three real-shaped examples (feature / bug / refactor).
