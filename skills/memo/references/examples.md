# Worked Examples

Three real-shaped `MEMORY.md` files. Mirror these structures.

---

## Example 1 — Feature build

```markdown
# MEMORY

> Created: 2026-06-08 · Last updated: 2026-06-10 14:22

## Goal
Add WebAuthn passkey login alongside existing email+password. Must coexist; users can have both. Ship behind feature flag `auth.passkeys`.

## Core Context
- `src/auth/strategies/` — one file per strategy; mirror `password.ts` for `passkey.ts`
- `users` table has no `credentials` column yet — migration `0042_passkey_credentials.sql` drafted, not applied
- We use `@simplewebauthn/server` v10 — RP ID must match cookie domain or registration silently fails
- Feature flag service: `src/flags.ts`, fetches from GrowthBook, cached 60s
- Frontend lives in separate repo `web-app`, not in scope

## Progress
- [x] Add `credentials` table migration (drafted)
- [x] Stub `passkey.ts` strategy
- [ ] Wire `/auth/passkey/register` route  ← cursor
- [ ] Wire `/auth/passkey/verify`
- [ ] Apply migration in staging
- [ ] Flip flag for internal users

## Decisions
- Store credentials in separate table, not JSON column on `users` — easier to revoke individually
- `@simplewebauthn/server` over `fido2-lib` — better TS types, active maintenance
- Skip backup-eligible flag for now — adds UX complexity, revisit post-launch

## Pitfalls
- RP ID `localhost` only works on `http://localhost:3000` exactly; `127.0.0.1` fails silently — use the dev cert from `scripts/dev-cert.sh`
- `navigator.credentials.create` rejects in iframe; preview env wraps the app in one — test on `dev.example.com` instead

## Open Questions
- Hardware keys too, or platform authenticators only? — waiting on @security
```

---

## Example 2 — Bug hunt

```markdown
# MEMORY

> Created: 2026-06-09 · Last updated: 2026-06-09 18:40

## Goal
Stop Stripe webhook from creating duplicate `Order` rows when Stripe retries. Reproduce on staging, fix, add regression test.

## Core Context
- Webhook handler: `src/webhooks/stripe.ts:handle`
- Idempotency was *intended* via `events.stripe_event_id` unique constraint — but constraint is missing in prod (added in dev migration `0038`, never deployed)
- Stripe retries with same `event.id` for ~3 days
- Order creation: `src/orders/create.ts` — wraps in transaction, but transaction commits before idempotency check ❗

## Progress
- [x] Reproduce locally with `stripe trigger payment_intent.succeeded --add`
- [x] Confirm prod missing constraint (`\d events` on prod replica)
- [ ] Apply migration in prod  ← cursor
- [ ] Re-order idempotency check before order create
- [ ] Backfill: dedupe existing duplicates from last 30d

## Decisions
- Fix in code (re-order check) AND in DB (constraint) — defense in depth

## Pitfalls
- Don't run dedupe backfill during business hours — locks `orders` for ~2min
- `stripe listen` in dev does NOT replay; must use `stripe events resend <id>`

## Open Questions
- _(none)_
```

---

## Example 3 — Refactor

```markdown
# MEMORY

> Created: 2026-06-05 · Last updated: 2026-06-10 09:11

## Goal
Move billing code out of `src/api/` into a self-contained `src/billing/` module. No behavior change. Cut import cycles.

## Core Context
- 23 files currently mix billing + API logic; tracked in `scratch/billing-files.txt` (not committed)
- Hard rule: no new `import from '../api/...'` inside `billing/` after move
- Tests use absolute imports — those are easy. Some legacy tests use `../../` — those break
- `madge --circular` currently reports 4 cycles, all in billing↔api

## Progress
- [x] Inventory files (23)
- [x] Move pure types (5 files)
- [x] Move services (8 files)
- [ ] Move handlers (10 files)  ← cursor
- [ ] Update legacy `../../` imports
- [ ] Run `madge --circular` — must be 0

## Decisions
- Keep one shared `types.ts` at top of billing/ — splitting would force re-imports everywhere
- Don't touch tests in this PR — separate follow-up

## Pitfalls
- VSCode "Move file" rewrites imports but misses `jest.mock('../../...')` strings — grep manually after each move

## Open Questions
- _(none)_
```

---

## Notes on shape

- All three are < 60 lines.
- Goal is always 1–3 sentences and verifiable.
- Pitfalls are *specific to this task*, never generic.
- The cursor (`← cursor`) makes resumption trivial.
