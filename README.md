# memo

> **Persistent working memory for long Agent sessions.**
> One tiny `MEMORY.md` per project that survives context compaction — so you stop re-burning tokens to rediscover what you already learned.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![skills.sh](https://img.shields.io/badge/skills.sh-compatible-brightgreen)](https://skills.sh)

English | [简体中文](./README.zh.md)

---

## The problem

Models have finite context. On any non-trivial feature, this happens:

1. You spend half the context window helping the model *understand* the codebase.
2. The conversation drifts long. Compaction fires. Hard-won understanding evaporates.
3. The next turn re-burns tokens rediscovering the same files, the same constraints, the same gotchas.

`CLAUDE.md` and `AGENTS.md` solve a different problem — *static* project conventions ("how we build here"). They don't help when the question is "what was I in the middle of, what did I just learn, and what already bit me twice today."

## The idea

`memo` introduces a third file: **`MEMORY.md`** — live, task-scoped, ruthlessly small working memory.

| File | Scope | Lifetime | Content |
|---|---|---|---|
| `CLAUDE.md` / `AGENTS.md` | Project-wide rules | Permanent | "How we do things here" |
| `README.md` | Human onboarding | Permanent | What the project is |
| **`MEMORY.md`** | **Active work in this project** | **While the work is in flight** | **Goal · core context · progress · decisions · pitfalls** |

The agent maintains it for you, throughout the session — automatically when you check off a milestone, hit a footgun, or make a decision worth remembering. In a fresh session, `/memo` reads it back in seconds and you pick up exactly where you left off.

> **One rule**: every line must save the next session more tokens than it costs. A 30-line memo of pure signal beats a 300-line journal.

---

## Installation

### Via Claude Code Plugin

```bash
/plugin marketplace add VinkyDev/memo
/plugin install memo
```

The plugin is installed from GitHub and registered as `memo`. Skills are invoked as `/memo:memo`.

To update or remove:

```bash
/plugin update memo
/plugin remove memo
```

### Via `skills.sh`

```bash
npx skills add https://github.com/VinkyDev/memo --skill memo 
```

---

## Usage

Just one command, with an optional trailing prompt:

```text
/memo
/memo <whatever you want to do next>
```

That's the whole interface. The skill figures out the rest:

- **First time in a project** → asks for the goal in one line, writes `MEMORY.md`, then handles your prompt.
- **`MEMORY.md` already exists** → loads it (silently — no file dump), acknowledges the current cursor, then handles your prompt.
- **As work progresses** → the skill auto-updates `MEMORY.md` whenever a milestone, decision, or pitfall worth remembering happens. You don't have to ask.

---

## Pair with /clear and /compact

`MEMORY.md` is the safety net that makes context resets cheap. Once it's in place:

- Run `/clear` or `/compact` freely — essential context lives in the file, not in the conversation.
- Shorter context = faster responses, lower cost, fewer compaction surprises.
- The memo pattern actively encourages aggressive context hygiene: compress early, compress often.

---

## How it stays small

`MEMORY.md` is enforced to be **tiny** by design:

- ~300-line soft cap; ~400-line hard cap (agent prunes before adding past that)
- Per-section caps (≤ 8 core context bullets, ≤ 5 decisions)
- Strict include/exclude rules baked into the skill — no code dumps, no chat logs, no restating project conventions

If you ask the agent to dump 200 lines of stack trace into the memory, it will push back. That's the point.

---

## What goes where

| Question | `CLAUDE.md` / `AGENTS.md` | `MEMORY.md` |
|---|---|---|
| "We use TypeScript with strict mode" | ✅ static convention | ❌ already known forever |
| "Auth lives in `src/auth/`" | ✅ project map | ❌ stable |
| "RP ID `localhost` only works on `:3000` exactly — `127.0.0.1` fails silently" | ❌ task-specific gotcha | ✅ pitfall, saves a future session 30 min |
| "We chose `@simplewebauthn/server` over `fido2-lib` because…" | ❌ decisions get lost in chat | ✅ decision + rationale |
| "Halfway done with the `register` route" | ❌ not a project rule | ✅ progress cursor |

**`CLAUDE.md` is the README for the project. `MEMORY.md` is the README for the work in progress.**

---

## License

[MIT](./LICENSE)
