# Authoring a high-signal MEMORY.md

This is the *philosophy* doc for the `memo` skill. Read it when you're unsure whether a piece of information belongs in MEMORY.md.

## The one rule

> **Every line must save the next session more tokens than it costs.**

If a line wouldn't save the model time, search effort, or a re-explanation in a fresh context, it doesn't belong.

## Signal vs. noise — examples

| ❌ Noise | ✅ Signal |
|---|---|
| "We are using TypeScript" | `tsconfig.json` has `strict: false` — don't enable, breaks 14 files |
| "Add error handling" | Errors from `parseTokens()` must surface as `AuthError`, not generic `Error` (front-end matches on `.code`) |
| "Refactor the auth module" | Goal: replace JWT with PASETO in `src/auth/`, keep `verify()` signature stable |
| Pastes 80 lines of code | `src/auth/jwt.ts:42` — token rotation lives here |
| "User wants login to work" | Login broken when `expiresAt` is null; `services/session.ts:88` returns `Infinity`, downstream `<` compare gives false |

## Sections, in priority order

1. **Goal** — without this, nothing else has meaning. Always present.
2. **Core Context** — the 3–8 facts that took the longest to learn. The single highest-ROI section.
3. **Current Progress** — keeps continuity across sessions; without it you re-plan every load.
4. **Pitfalls** — protects future-you from repeating past-you's mistakes.
5. **Decisions** — prevents the model from "improving" something on purpose.
6. **Open Questions** — un-blockable items waiting on the human.
7. **Scratch** — anything else; ruthlessly pruned.

## When to prune

Prune on every `update`. Pruning rules:

- **Scratch entry > 24h old?** → promote to a real section, or delete.
- **Pitfall no longer applicable?** (e.g., bug fixed) → delete.
- **Decision now obvious from the code?** → delete.
- **Progress item finished and no longer informative?** → keep one line, drop sub-bullets.
- **Total > 200 lines?** → force a prune pass before adding anything new.

## What MEMORY.md is NOT

- Not a journal. No "today I tried...".
- Not a spec. No "the system shall...".
- Not documentation. No examples for future readers.
- Not a TODO list for the human. Use issues for that.
- Not a paste buffer. Use scratch files in `/tmp` for that.

## Comparison cheat-sheet

```
CLAUDE.md / AGENTS.md  →  STATIC, project-wide, "how we build"        (rarely edited)
README.md              →  STATIC, human-facing, "what this is"        (rarely edited)
MEMORY.md              →  LIVE, task-scoped, "what I'm doing & learned" (edited daily)
```

If the same content fits in two of these, it belongs in the more permanent one. Memory should *complement*, never *duplicate*.
