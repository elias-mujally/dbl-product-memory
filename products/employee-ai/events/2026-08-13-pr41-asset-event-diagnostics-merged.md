# 2026-08-13 — PR #41 Asset-Event Diagnostics merged

## Context

After PR #40 was merged, a controlled Desktop Chrome Production attempt progressed further than the Android attempt:

```text
sdk_loading
→ sdk_loaded
→ prepare_started
→ prepare_ready
→ sdk_ready
→ login_invoked
→ callback_received
→ authorization_code_received
→ asset_event_missing
```

The authorization code was successfully received, but no accepted WhatsApp asset event entered the completion gate before the existing 12-second timeout. No completion started and no credential/account-scope mutation occurred.

The remaining diagnostic gap was that ignored/rejected `postMessage` asset events were mostly silent, so Production could not distinguish whether Meta emitted nothing or whether DBL rejected an event.

## PR #41 scope

PR #41 added diagnostic-only, privacy-safe Embedded Signup asset-event classifications without changing acceptance rules, origin allowlists, completion semantics, timeout behavior, PR #40 browser isolation, PR #38 lifecycle safety, or PR #39 outbound readiness.

Diagnostic vocabulary:

- `asset_event_origin_rejected`
- `asset_event_malformed`
- `asset_event_type_unsupported`
- `asset_event_category_unsupported`
- `asset_event_fields_invalid`
- `asset_event_attempt_fenced`
- `asset_event_accepted`

The telemetry contains only static allowlisted categories. It does not log raw origins, raw payloads, MessageEvents, authorization codes, access tokens, credential references, WABA IDs, phone IDs, customer/Contact/workspace identifiers, popup URLs, or fragment/context IDs.

## Final independent review

Reviewed head:

`e58fd33ba081130b132d7deb64a7be58e775496c`

Result:

- High findings: `0`
- Medium findings: `0`
- Low finding: no explicit mounted-handler telemetry transport failure-injection test; correctness remains independent of telemetry delivery
- Production risk: `2/10 Low`
- focused tests: `134/134`
- full Vitest: `494 passed, 5 skipped`
- database/Foundation checks: passed
- PR #40 multi-browser isolation: passed
- PR #38 T1–T7 + completion contract: passed
- PR #39 readiness/outbound regressions: passed
- Vercel Preview: READY

## Merge

- PR: `#41`
- reviewed head: `e58fd33ba081130b132d7deb64a7be58e775496c`
- squash SHA: `e7634cc3e35567f877515c5c72a8ecfe22b11e03`
- merge method: squash with exact-head protection
- GitHub confirmed successful merge

## Current operational decision

The next step is **not** webhook cutover and not a general reconnect loop.

After Vercel Production is confirmed READY on squash SHA `e7634cc3e35567f877515c5c72a8ecfe22b11e03`, authorize exactly one controlled Desktop Embedded Signup diagnostic attempt using an owner/admin session.

Purpose:

- determine whether Meta emits an asset event;
- if emitted, identify the privacy-safe rejection/acceptance category;
- preserve the existing connection/account scope and legacy credential unless a separately authorized completion is explicitly allowed.

Meta webhook routing must remain on legacy Cloud Run during this diagnostic stage.

Contacts PR 2 and PR C remain unstarted/blocked.
