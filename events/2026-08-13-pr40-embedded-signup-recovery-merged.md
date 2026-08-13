# 2026-08-13 — PR #40 Embedded Signup Production Diagnostics & Recovery merged

## Summary

PR #40 was squash-merged after multiple independent adversarial review cycles closed all High and Medium findings on the exact reviewed head.

Application repository:

`elias-mujally/dbl-employee-ai`

PR:

`#40`

Reviewed head:

`5488ce4485e141323e3f879e1ef28bced46251cf`

Squash SHA:

`a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`

## Why PR #40 existed

Production Embedded Signup had been observed to reach synchronous `FB.login()` successfully but return a callback without `authResponse.code`, which the application collapsed into a generic `authorization_incomplete` outcome. The exact external Meta/browser cause remained unproven.

The goal of PR #40 was therefore not to guess the Meta-side cause, but to make the flow diagnosable, recoverable, and safe enough for a later controlled same-number reconnect.

## Main improvements

- privacy-safe Embedded Signup lifecycle vocabulary and diagnostics;
- more precise callback/error taxonomy without exposing provider payloads;
- bounded recovery when expected asset events do not arrive;
- synchronous `FB.login()` preserved inside the original user click;
- truthful recovery UX in Arabic and English;
- testing-stage Meta notice reduced to contextual owner/admin information;
- legacy connected state continues to guide owner/admin toward reconnecting the same WhatsApp number;
- successful synthetic completion remains compatible with PR #39 modern `gcp-sm://` readiness;
- no webhook, Meta configuration, Cloud Run traffic, database schema, or Production data changes.

## Important browser-isolation findings discovered during review

Two separate Medium findings were discovered and fixed before merge.

### 1. Stale Meta FINISH event could cross attempts

Meta `WA_EMBEDDED_SIGNUP` FINISH postMessages do not echo a DBL attempt nonce. A delayed asset event from attempt A could therefore be associated with attempt B if both attempts were allowed in the same browser context.

The first fix isolated retries into new `noopener noreferrer` contexts and fenced old attempts.

### 2. Fragment-keyed fence was bypassable

The first isolation design persisted the consumed marker under a URL-fragment-derived key. Loading a fresh recovery fragment in the original tab could therefore restore launch eligibility while the old Meta popup still retained the original browsing context's WindowProxy.

The final fix replaced this with a browsing-context-wide fixed sessionStorage marker:

`dbl_whatsapp_embedded_signup_context_consumed`

Final invariant:

> One browser browsing context may launch Embedded Signup at most once. Fragment changes, fragment removal, reload, remount, SPA navigation, or manually pasting a recovery URL into the same tab cannot restore launch eligibility.

The URL fragment is now non-authoritative UX metadata only.

## Fail-closed storage semantics

Before calling `FB.login()` the browser synchronously establishes and reads back the consumed marker.

If sessionStorage read/write/persistence is unavailable, launch fails closed and Meta is not invoked.

This preserves correctness even though storage-disabled browsers may need clearer UX later.

## Cross-browser verification

Real browser isolation tests passed across:

- Chromium: 5/5
- Firefox: 5/5
- WebKit: 5/5

They cover same-tab fragment replacement/removal/reload, genuine noopener contexts, opener isolation, stale signal routing, A→B→C recovery, and storage-failure behavior.

## Final independent review

Exact reviewed head:

`5488ce4485e141323e3f879e1ef28bced46251cf`

Result:

- High findings: 0
- Medium findings: 0
- Production merge risk: Low (2/10)
- Focused Embedded Signup: 57 passed
- Vitest: 489 passed, 5 skipped
- Authenticated E2E: 17 passed
- Multi-browser isolation: 15 passed
- pgTAP: 466 passed
- account-scope concurrency T1–T7: passed
- completion service contract: 5 passed
- Supabase reset / Contacts upgrade / lint / typecheck / format / build / git diff check: passed
- Vercel Preview: READY
- focused secret/PII scan: passed

## Merge

PR #40 was marked Ready for Review and squash-merged with exact-head protection.

Squash SHA:

`a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`

GitHub confirmed successful merge.

## Remaining Low debt

- recovery intentionally opens an additional tab/window;
- browsers without usable sessionStorage fail safely but do not yet have dedicated storage-specific guidance;
- isolation tests exercise production browser helpers rather than a fully mounted React wizard;
- real Meta popup/callback behavior still requires the separately authorized controlled Production reconnect.

## Operational status after merge

PR #40 being merged does NOT authorize webhook cutover.

The next approved step is a controlled Production same-number reconnect after:

- Vercel Production is READY on squash SHA `a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`;
- current Embedded Signup config ending `1674` is reconfirmed;
- existing connection/account scope is inspected read-only;
- an authenticated owner/admin session is used.

During reconnect:

- reconnect the same receiving number only;
- stop on any different-number conflict;
- stop on ambiguous completion;
- do not blindly retry;
- verify the credential reference transitions to modern `gcp-sm://...`;
- verify PR #39 readiness becomes `ready`;
- verify account scope and historical customers, Contacts, identities, conversations, and messages remain unchanged.

Meta inbound webhook routing remains on the legacy Cloud Run runtime until a separate controlled cutover is reviewed and authorized.

Contacts PR 2 and PR C remain unstarted.