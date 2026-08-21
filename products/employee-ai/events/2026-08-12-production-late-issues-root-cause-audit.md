# 2026-08-12 — Production late-discovered issues root-cause audit

## Context

بعد دمج PR #38 وإجراء اختبار يدوي حقيقي على Production، ظهرت أربع مشكلات عملية في WhatsApp وEmbedded Signup وOnboarding. تم تنفيذ read-only Production Bug Audit على `main` الحالي دون تعديل ملفات أو بيانات إنتاج أو إعدادات Meta/Vercel/Supabase.

## Source of truth inspected

- application `main`: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- production deployment: `dpl_5X9Jtqsb5qYkneNTsL4Dr4ZZVcgB`
- production deployment SHA: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- Vercel status: `READY`

The local checkout was older/dirty and was not used as source of truth.

---

## LDI-001 — Static WhatsApp acknowledgement still appears

### Status

**High — strong evidence, final external routing target still needs read-only confirmation.**

### Observed behavior

Inbound WhatsApp messages receive the retired test acknowledgement:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

### Audit findings

- The exact acknowledgement text does **not** exist on current `main`.
- Historical code introduced this reply as an early receipt-test responder.
- That responder was later deleted when the grounded AI workflow replaced it.
- Production Supabase recorded inbound webhook/message activity during the relevant period.
- Vercel showed no `/api/webhooks/whatsapp` invocation for that same period.
- No current automatic outbound request or AI run was created at inbound receipt time.

### Current conclusion

Strong evidence indicates Meta production webhook traffic is still reaching an **older Cloud Run runtime/revision** containing the retired receipt responder instead of the canonical Vercel webhook runtime.

This is not yet promoted to fully proven external-routing root cause until the active Meta callback URL or Cloud Run request logs are confirmed read-only.

### AI cost correction

The static acknowledgement itself does **not** call the AI provider and does not consume AI tokens.

Two later AI events were separate:

- one deterministic safe-conversation run with estimated `0/0` tokens and no provider request identifier;
- one manually triggered conversation-generation run that failed with `generation_failed`.

There is currently no evidence that the static acknowledgement and AI generation both ran automatically for the same inbound receipt.

---

## LDI-002 — Embedded Signup returns authorization incomplete

### Status

**Medium — exact Meta/browser root cause not yet proven.**

### Proven application flow

```text
page load
→ SDK ready
→ FB.init
→ prepared server state ready
→ button enabled
→ synchronous FB.login invoked
→ callback returned without authResponse.code
→ authorization_incomplete
→ no completion action / no DB mutation
```

### What is not yet proven

The audit did not establish why Meta returned without an authorization code.

Leading hypotheses requiring authenticated Production reproduction include:

- Production JavaScript SDK domain/origin authorization issue;
- Meta login/session/cookie state;
- browser popup/privacy behavior.

PR #38 did not modify browser SDK sequencing.

### Testing-mode banner decision

Do not remove the Meta testing-mode notice entirely while external/commercial Meta approvals remain incomplete.

The approved UX direction is to reduce it to a low-prominence contextual owner/admin notice during connection/reconnection rather than a persistent warning suggesting the product is broken.

Remove it only when release-stage configuration is intentionally `approved` and the corresponding Meta approvals are actually complete.

---

## LDI-003 — Manual outbound WhatsApp sending fails

### Status

**High — root cause proven.**

### Root cause

The active Production WhatsApp connection still references the legacy credential location:

`env:WHATSAPP_ACCESS_TOKEN`

The current Vercel runtime expects the modern workspace credential path backed by Secret Manager/WIF. The legacy provider configuration is unavailable/incomplete in the current runtime.

### Failure boundary

```text
manual send action
→ authorization/routing succeed
→ outbound request created
→ connection loaded
→ provider construction
→ legacy credential configuration unavailable
→ provider_configuration_missing
→ permanent_failure
→ no Meta Graph API request
```

### Production evidence

Two manual outbound requests/attempts failed with:

- attempt status: `permanent_failure`
- safe category: `provider_configuration_missing`
- no provider HTTP status
- no provider error response

### Connection health

- connection status: connected
- receiving account scope: present and internally consistent
- credential reference: present but legacy environment-based
- modern Secret Manager credential: not referenced
- no evidence PR #38 credential lineage corrupted the connection

Blind retries should not be used. The connection should transition through a safe **same-number Embedded Signup** or another explicitly approved credential transition.

---

## LDI-004 — Onboarding response mode reaches impossible state

### Status

**High — root cause proven as UI/state-machine inconsistency.**

### Production state observed

- persisted `automatic_replies_enabled = true`
- onboarding employee test completed
- test result `grounded = false`
- activation incomplete

### Root cause

The page derives Automatic eligibility from current readiness but derives the checked radio value from persisted AI settings.

This can render:

```text
Automatic = checked + disabled
Review-only = unchecked
```

Disabled HTML controls are omitted from form submission, so activation receives no response mode and returns:

`mode_invalid`

The radio controls are uncontrolled, allowing reload/server-error reconstruction into the same contradictory state.

### Important policy finding

Review-only is valid at the database activation boundary and should not require Automatic-mode readiness when explicitly selected.

The observed failure is primarily state serialization/UI authority drift, not a review-only policy rejection.

### Employee-test finding

The employee test completed but produced `grounded=false`; this was not a transport failure.

The database currently lacks enough safe diagnostic detail to distinguish among insufficient approved knowledge, unsupported answer, grounding validation rejection, or another review-required result.

---

## Cross-issue architecture finding

The audit found a broader **split-runtime / legacy integration state**:

```text
legacy Meta webhook target / older Cloud Run
├─ retired static acknowledgement remains visible          [LDI-001]
└─ canonical Vercel automatic workflow may not receive inbound event

legacy DB credential reference
├─ older runtime can apparently send receipt
└─ current Vercel outbound provider cannot construct       [LDI-003]

Embedded Signup authorization incomplete
└─ blocks the normal same-number credential transition     [LDI-002 → keeps LDI-003 unresolved]

AI settings/readiness authority drift
└─ impossible response-mode onboarding state               [LDI-004]
```

LDI-001 and LDI-003 therefore share a broader legacy/current runtime split.

LDI-002 prevents the preferred same-number reconnection path that could transition the active connection to the modern credential store.

LDI-004 is a separate onboarding state-machine problem.

None of the four examined issues was introduced by PR #38.

---

## Additional Medium findings from audit

1. **Outbound observability:** multiple credential/provider failures collapse into `provider_configuration_missing`, reducing diagnosis quality.
2. **AI observability:** unexpected provider failures collapse into `generation_failed`, preventing authoritative cost/failure-stage diagnosis.
3. **Simple-reply semantics:** product copy can imply deterministic conversational replies operate automatically while review-only mode still prevents all automatic sending.
4. **Readiness authority drift:** onboarding UI, activation RPCs, and persisted AI settings use partially different readiness/state authorities, allowing contradictory states beyond the single radio bug.

---

## Approved repair direction

Do **not** build one combined all-in-one repair PR.

### First gate — read-only external routing confirmation

Before implementing repairs or changing production routing, confirm read-only:

- active Meta webhook callback target;
- whether it points to the old Cloud Run runtime;
- whether Cloud Run request logs confirm current inbound delivery;
- whether canonical Vercel webhook receives none of those events.

No Meta configuration mutation during this gate.

### PR A — WhatsApp credential/runtime readiness

Scope:

- provider error taxonomy;
- connection-health representation;
- modern credential readiness;
- safe same-number reconnection readiness;
- manual/automatic outbound diagnostics;
- no production webhook configuration change inside this PR.

### PR B — Embedded Signup production diagnostics and recovery

Scope:

- safe SDK/login callback telemetry;
- Production-domain reproduction;
- precise authorization-incomplete diagnostics;
- contextual testing-mode notice;
- no real provider asset mutation in automated tests.

### Operational cutover

Only after PR A + PR B are verified:

- explicitly approve Meta webhook callback transition from old runtime to canonical Vercel webhook;
- verify signed delivery and idempotency;
- verify static receipt is gone;
- then retire/disable the old responder/runtime as appropriate.

### PR C — Onboarding response-mode state machine

Scope:

- controlled/recoverable response-mode selection;
- shared readiness authority;
- never submit a disabled checked control as the only selected state;
- preserve selection across errors/reload;
- safe employee-test outcome categories;
- Arabic/English/RTL/mobile/accessibility coverage.

PR C is technically independent, but WhatsApp High issues take priority.

---

## Current execution decision

- **GO** for targeted repair work using the split above.
- **NO-GO** for Contacts PR 2 until these production issues are repaired and verified.
- **NO-GO** for an all-in-one repair PR.
- **NO-GO** for immediate production webhook cutover until the active callback/runtime target is confirmed read-only.

### Immediate next task

**Read-only Meta webhook / Cloud Run target confirmation.**

This confirmation should establish whether the retired receipt responder is still the active inbound production path before any external routing change is authorized.
