# 2026-08-13 — PR #42 malformed-shape diagnostics merged

## Summary

PR #42 — Embedded Signup malformed-shape diagnostics was independently reviewed and squash-merged.

Reviewed head:

`e7e013602c1f9983ed3f9ba4e29eb3de1ded36d0`

Squash SHA / current main after merge:

`ede862e60bf1ace67a1e3fbb49d1e3f3d4bc7caf`

The PR refines the coarse `asset_event_malformed` lifecycle category into four privacy-safe structural categories only:

- `asset_event_malformed_json`
- `asset_event_malformed_null`
- `asset_event_malformed_array`
- `asset_event_malformed_primitive`

The review confirmed this is label-only behavior. Origin acceptance, parser acceptance/rejection, FINISH/CANCEL/ERROR semantics, identifier validation, timeout, completion gating, browser isolation, and exactly-once behavior remain unchanged.

## Why this exists

A controlled Desktop Production attempt after PR #41 produced:

```text
sdk_loading
→ sdk_loaded
→ prepare_started
→ prepare_ready
→ sdk_ready
→ login_invoked
→ asset_event_malformed
→ callback_received
→ authorization_code_received
→ asset_event_missing
```

This proved that at least one allowed-origin postMessage reached DBL during the Embedded Signup attempt, but the prior telemetry could not distinguish whether it was invalid JSON, null, array, or primitive data.

A read-only audit concluded that the current parser is not proven wrong. The strongest current hypothesis is that the malformed message may be unrelated Facebook SDK/OAuth coordination traffic rather than a WhatsApp `FINISH` event. A second possibility is that authorization completed without the user flow reaching final WABA/phone asset selection. No Meta configuration or parser acceptance change is justified yet.

## Review / risk

Independent review result:

- High findings: 0
- Medium findings: 0
- Remaining Low: minor test-depth gap only
- Production merge risk: 1/10

Validation included:

- focused malformed-shape tests: 24 passed
- full Vitest: 495 passed, 5 skipped
- authenticated E2E: 17/17
- multi-browser isolation: 15/15
- pgTAP: 466
- account-scope T1–T7: passed
- completion contract: 5/5
- Vercel Preview: READY
- privacy/secret inspection: passed

## Operational status

No real reconnect, Meta asset mutation, webhook cutover, Cloud Run traffic change, or credential transition was performed as part of PR #42.

The next approved step is:

1. wait for Vercel Production to become READY on squash SHA `ede862e60bf1ace67a1e3fbb49d1e3f3d4bc7caf`;
2. perform exactly one controlled Desktop Embedded Signup attempt;
3. capture the ordered privacy-safe lifecycle categories;
4. stop and analyze whether the malformed signal is JSON/null/array/primitive.

Meta webhook routing remains on legacy Cloud Run and cutover remains NO-GO.
