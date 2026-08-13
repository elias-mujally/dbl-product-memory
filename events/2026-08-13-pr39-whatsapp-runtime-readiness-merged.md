# PR #39 — WhatsApp Credential / Runtime Readiness — MERGED

Date: 2026-08-13

## Merge

PR #39 was independently reviewed on exact head:

`e7a3d50b7d2b2328cd6bfc7d2c91231c6efc2d7b`

Final review result:

- High findings: 0
- Medium findings: 0
- Production risk: Low

The PR was marked Ready for Review and squash-merged with exact-head protection.

Squash SHA:

`7f5d78ee3441be4cce8c8671992f08cb98b43717`

## What PR #39 changes

PR #39 establishes a server-authoritative WhatsApp outbound readiness model without persisting a new readiness column.

It distinguishes, among other states:

- modern Secret Manager-ready credentials
- legacy `env:WHATSAPP_ACCESS_TOKEN` credentials requiring same-number reconnect
- missing/malformed/unknown credential references
- WIF/auth/store/secret failures
- unsupported/invalid provider configuration
- incomplete account scope

The legacy Production connection remains historically connected, but the current Vercel runtime no longer presents it as outbound-ready.

## Manual outbound safety

The independent review initially found a Medium issue: manual readiness was checked after durable outbound reservation, so repeated page reloads with fresh request IDs could create duplicate failed outbound records and consume quota even though Meta was never called.

The correction moved an authoritative readiness check before reservation while retaining the late provider/readiness check for TOCTOU protection.

Final verified order:

```text
authenticated workspace
→ trusted conversation
→ trusted WhatsApp connection
→ early readiness check
→ durable reservation only if eligible
→ late readiness/provider check
→ send
```

Two distinct manual request IDs against the same legacy connection were verified to produce:

- 0 reservation RPC calls
- 0 outbound requests
- 0 outbound messages
- 0 outbound attempts
- 0 quota consumption
- 0 Meta/provider calls

The reviewed draft remains preserved.

## Automatic outbound

Automatic sending retains deterministic request identity and the same final readiness boundary.

For blocked legacy credentials:

- no Meta call
- no retry loop
- no duplicate request
- generated content remains preserved

## UX

WhatsApp connection state and outbound runtime readiness are represented separately.

A legacy connection may remain visibly connected while owners/admins receive guidance to reconnect the same WhatsApp number before sending from the current runtime.

The UI does not claim the connection is disconnected and does not recommend blind retries.

## Security / scope

PR #39 introduced:

- no schema migration
- no credential copy/migration
- no credential deletion
- no Meta callback/subscription change
- no Cloud Run traffic change
- no Contacts PR 2 work
- no PR B work

PR #38 account-scope lifecycle protections remain intact.

## Exact-head validation

Final reviewed head passed:

- Foundation checks
- lint/typecheck/format/build
- Vitest: 481 passed, 5 skipped
- Supabase reset
- Contacts upgrade harness
- pgTAP: 466 passed
- PR #38 T1–T7 concurrency
- completion service contract: 5 passed
- authenticated E2E: 12 passed
- Vercel Preview
- focused secret/PII scan

## Current status after merge

PR #39 is merged, but Production still cannot perform modern outbound delivery until an owner/admin successfully reconnects the same WhatsApp number through the reviewed Embedded Signup flow.

Meta inbound webhook still targets the legacy Cloud Run responder. No webhook cutover was performed.

Next engineering task:

**PR B — Embedded Signup Production diagnostics and recovery.**

Contacts PR 2 remains blocked until messaging foundation repair and controlled webhook cutover are production-verified.
