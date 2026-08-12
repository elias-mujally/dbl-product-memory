# PR #38 — WhatsApp Account-Scope Lifecycle Guard — MERGED

**Date:** 2026-08-12

## Status

PR #38 was squash-merged after repeated independent adversarial reviews closed all High and Medium findings.

- PR: `#38`
- Final reviewed head: `1325412e13fbf8f8925127ec231d744f2a1d0caa`
- Squash SHA: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- Merge method: squash
- Merge protection: exact reviewed head SHA
- Production verification: **pending at the time of this memory update**
- Contacts PR 2: **not started**

## Why PR #38 existed

While planning Contacts PR 2, the technical audit discovered a lifecycle contradiction:

- Contacts Foundation treats the receiving WhatsApp `phone_number_id` as an immutable historical account scope for channel identity.
- Existing Embedded Signup completion could reuse a `whatsapp_connections` row and replace its `phone_number_id` in place.

If left unresolved, a connection with customer history could silently change provider account scope and invalidate the meaning of historical Contact/channel identity relationships.

The approved product policy became:

> Once a WhatsApp connection has customer or canonical Contact identity history, its receiving `phone_number_id` may not be replaced in place. Reconnecting the same number remains allowed. A different number must eventually use a separate connection/account scope, with no automatic Contact merge.

The current single-connection MVP does not yet support a second independent connection, so different-number replacement with history is rejected truthfully rather than pretending a recovery flow already exists.

## Final account-scope guard behavior

- New connection: existing behavior unchanged.
- Same receiving `phone_number_id`: allowed, including reconnect/credential refresh.
- Different `phone_number_id` with no customer or canonical identity history: current in-place behavior remains allowed.
- Different `phone_number_id` with customer or Contact identity history: rejected transactionally.
- Existing Contacts, channel identities, legacy mappings, customers, conversations and messages are preserved.
- No Contacts PR 2 dual-write was introduced.

History detection uses existing customer history and canonical Contact identity history so historical scope cannot be rewritten after the legacy/canonical transition.

## Major correctness problems found and fixed during review

PR #38 required several review/fix cycles. These are intentionally preserved in Product Memory because they document important distributed-systems lessons and operational guarantees.

### 1. Stale credential predecessor race

Initial cleanup logic read the previous credential before the authoritative database lock. Concurrent same-number completions could both observe the same predecessor and leave an intermediate credential orphaned.

Final design:

- credential lineage is captured transactionally while the connection row is locked;
- the authoritative completion function returns the exact credential reference replaced by that committed mutation;
- cleanup targets only that exact predecessor.

### 2. Service-level concurrency coverage gap

Early concurrency tests proved database serialization but did not prove the complete database → service → credential-store contract.

Final coverage includes real local RPC boundaries, deterministic fake credential storage and concurrent service-level completion behavior.

### 3. Misleading recovery UX

Early copy told users to connect a different number as a separate connection even though multi-connection support does not exist yet.

Final copy states truthfully that the same number can be reconnected and separate-number support is not currently available.

### 4. Ambiguous committed response could delete the active credential

Critical distributed-systems failure mode:

```text
DB commit succeeds
→ candidate credential becomes active
→ RPC/transport response is lost
→ application assumes rollback
→ application deletes the active credential
```

Final design adds authoritative completion reconciliation.

Outcomes:

- `ACTIVE`: candidate is confirmed active, preserved, and normal success is returned.
- `NOT_COMMITTED`: candidate cleanup is allowed only after authoritative durable non-commit fencing.
- `INDETERMINATE`: candidate is preserved and the result remains explicitly unknown.

A raw timeout/exception/response loss is never considered proof of rollback.

### 5. NOT_COMMITTED was initially only a point-in-time observation

A delayed original completion could still commit after reconciliation said `NOT_COMMITTED` and after the candidate was deleted.

Final fix:

- reconciliation locks the signup session;
- when non-commit is authoritatively proven, it atomically transitions the existing session from `completing` to terminal `failed`;
- terminal reason: `whatsapp_completion_reconciled_not_committed`;
- only after that durable fence commits may candidate cleanup occur;
- delayed or retried completion must require `status='completing'`, therefore it cannot commit after the fence.

This converts `NOT_COMMITTED` from a snapshot into a durable terminal assertion.

## Important database/runtime mechanisms

PR #38 ultimately includes the account-scope guard, credential lineage, authoritative reconciliation and durable non-commit fencing through forward-only migrations.

Key properties retained:

- server/service-controlled mutation boundaries;
- hardened SQL functions;
- empty `search_path` where applicable;
- explicit schema qualification;
- no browser write privilege expansion;
- no RLS weakening;
- backward-compatible legacy completion RPC behavior;
- rolling-deployment compatibility;
- no Contact merge or Contact ingestion dual-write yet.

## Concurrency and failure coverage

Final reviewed coverage includes scenarios T1–T7 and service-contract tests, including:

- first customer history racing a different-number reconnect;
- reconnect completing before waiting ingestion;
- competing different-number reconnects;
- concurrent same-number completions with exact credential lineage;
- realistic retry/idempotency paths;
- delayed completion after durable NOT_COMMITTED fence;
- concurrent reconciliation attempts;
- commit succeeded / response lost;
- confirmed non-commit cleanup;
- indeterminate reconciliation;
- credential cleanup failure.

Final exact-head validation reported:

- Vitest: `464` passed
- pgTAP: `466` passed
- concurrency T1–T7: passed
- service-contract tests: `5` passed
- Supabase reset: passed
- Contacts migration-upgrade harness: passed
- authenticated E2E: passed
- production build: passed
- Vercel Preview: ready
- secret/PII scan: passed

## Final independent review

Exact reviewed head:

`1325412e13fbf8f8925127ec231d744f2a1d0caa`

Final outcome:

- High findings: `0`
- Medium findings: `0`
- Production risk before merge: **Low**
- PR marked Ready for Review
- exact head approved for protected squash merge

Accepted Low operational debt:

- after a reconciled ACTIVE result with a lost lineage response, an obsolete predecessor may require manual protected cleanup rather than unsafe guessing;
- cleanup failures are observable but do not yet have a durable automatic retry queue;
- Meta-side activation may precede DBL account-scope rejection, while DBL state remains fail-closed.

## Current stopping point

PR #38 is merged, but Product Memory must not mark it fully production-closed until post-merge Production verification confirms:

- all PR #38 migrations applied exactly once and in order;
- Vercel Production is READY on squash SHA `57052f665150849c1040bc173a8aef3bb9b02ab8`;
- no migration/RPC/runtime/5xx error cluster;
- existing WhatsApp connection/history remains intact;
- credential access remains healthy;
- account-scope guard and role restrictions remain correct;
- no Contacts PR 2 work has started.

After Production verification passes, PR #38 may be marked fully closed and Contacts PR 2 planning/implementation may resume.
