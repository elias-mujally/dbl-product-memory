# الحالة الحالية

آخر تحديث: **2026-08-13**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp integration يمر حاليًا بمرحلة إصلاح Production foundation قبل متابعة Contacts PR 2.

تم إغلاق **Contacts PR 1 — Foundation** بالكامل، وتم دمج **PR #38 — WhatsApp Account-Scope Lifecycle Guard**، ثم **PR #39 — WhatsApp Credential / Runtime Readiness**، وتم الآن كذلك دمج **PR #40 — Embedded Signup Production Diagnostics & Recovery** بعد مراجعة مستقلة نهائية دون High أو Medium findings.

الحالة الحالية:

> **PR #40 merged. Next approved operational step: controlled Production same-number reconnect to transition the legacy credential to modern Secret Manager. Meta webhook cutover remains NO-GO until reconnect/readiness verification succeeds. Contacts PR 2 and PR C remain blocked/unstarted.**

---

## Production الحالي

- Production app: `https://dbl-employee-ai.vercel.app`
- current GitHub `main` after PR #40 merge: `a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`
- PR #40 reviewed head: `5488ce4485e141323e3f879e1ef28bced46251cf`
- PR #40 squash SHA: `a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`
- PR #40: **MERGED via squash**
- Meta webhook routing: **unchanged; still legacy Cloud Run**

References:

- `events/2026-08-13-pr39-whatsapp-runtime-readiness-merged.md`
- `events/2026-08-13-pr40-embedded-signup-recovery-merged.md`

---

## Confirmed split-runtime

Read-only Production confirmation on 2026-08-12 proved:

```text
Meta inbound webhook
→ legacy Cloud Run service/revision
→ historical static receipt responder

Current DBL UI / server actions
→ Vercel current main
→ modern outbound / AI / Embedded Signup code
```

Classification:

`CONFIRMED_LEGACY_CLOUD_RUN`

The active Meta WhatsApp callback is still the old Cloud Run endpoint, not the canonical Vercel webhook.

Cloud Run service `dbl-employee-ai-git` still sends 100% traffic to legacy revision `00024`, which contains the retired static responder:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

Reference:

`events/2026-08-12-meta-webhook-legacy-cloud-run-confirmed.md`

---

## PR #39 — WhatsApp Credential / Runtime Readiness — MERGED

PR #39 made outbound readiness server-authoritative and separated provider connection status from actual runtime send readiness.

Key behavior:

- legacy `env:WHATSAPP_ACCESS_TOKEN` connection remains Connected historically but is not considered outbound-ready;
- owner/admin are guided to reconnect the same number;
- manual permanent readiness failures block before durable reservation;
- repeated manual attempts do not create duplicate outbound requests/messages/attempts or consume quota;
- late readiness/provider validation remains for TOCTOU protection;
- automatic outbound remains fail-closed for legacy credentials;
- modern `gcp-sm://` credentials classify as ready when accessible.

Reviewed head:

`e7a3d50b7d2b2328cd6bfc7d2c91231c6efc2d7b`

Squash SHA:

`7f5d78ee3441be4cce8c8671992f08cb98b43717`

---

## PR #40 — Embedded Signup Production Diagnostics & Recovery — MERGED

### الهدف

PR #40 does not guess the external Meta/browser cause of the previously observed callback-without-code behavior. Instead it makes Embedded Signup diagnosable, recoverable, and browser-safe enough for a separately controlled same-number reconnect.

### Browser lifecycle / diagnostics

- privacy-safe lifecycle stages added for SDK, preparation, login, callback, authorization, asset event, completion, failure, and unknown outcomes;
- cancellation, Meta error, missing code, missing asset event, popup uncertainty, completion failure, and ambiguous completion are distinguished without exposing provider payloads;
- `FB.login()` remains synchronous inside the genuine user click;
- missing asset-event wait is bounded rather than indefinite;
- authorization codes remain in memory only and are never logged/persisted client-side.

### Cross-attempt isolation

Independent review discovered two Medium issues before merge:

1. delayed Meta FINISH events could cross attempts because Meta does not echo a DBL attempt nonce;
2. the first isolation design keyed the launch fence by URL fragment, allowing same-tab fragment changes to restore launch eligibility.

Final invariant:

> **One browsing context may launch Embedded Signup at most once.**

The authoritative browser fence uses one fixed sessionStorage marker:

`dbl_whatsapp_embedded_signup_context_consumed`

It is independent of fragment, actor, workspace, session, provider asset, or URL.

After the first launch, the same browsing context remains fenced across:

- fragment replacement/removal;
- reload;
- React remount;
- SPA/navigation return;
- manually pasting a fresh recovery URL into the same tab.

Recovery requires a genuinely new `noopener noreferrer` browsing context.

The URL fragment is now non-authoritative UX metadata only.

### Storage safety

Before `FB.login()` the marker is synchronously written and read back.

If sessionStorage is unavailable or persistence cannot be proven, launch fails closed and Meta is not invoked.

### Cross-browser validation

Real-browser isolation passed:

- Chromium: 5/5
- Firefox: 5/5
- WebKit: 5/5

A→B→C recovery, stale FINISH/callback isolation, same-tab fragment bypass, reload, and noopener storage isolation all passed.

### Final review

Reviewed head:

`5488ce4485e141323e3f879e1ef28bced46251cf`

Result:

- High: **0**
- Medium: **0**
- Production merge risk: **Low (2/10)**
- Focused Embedded Signup: 57 passed
- Vitest: 489 passed, 5 skipped
- authenticated E2E: 17 passed
- multi-browser isolation: 15 passed
- pgTAP: 466 passed
- T1–T7: passed
- completion service contract: 5 passed
- Vercel Preview: READY

### Merge

- squash SHA: `a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`
- merge method: squash with exact-head protection
- GitHub confirmed successful merge

### Remaining Low debt

- recovery uses an additional tab/window;
- storage-disabled browsers fail safely but lack dedicated storage-specific guidance;
- browser-isolation tests use production helpers rather than a fully mounted wizard;
- real Meta popup/callback behavior still needs controlled Production verification.

---

## Late-discovered production issues

Reference:

`LATE_DISCOVERED_ISSUES.md`

### LDI-001 — Static WhatsApp acknowledgement

**Root cause: CONFIRMED.**

Legacy Cloud Run still owns inbound Meta webhook traffic and still contains `receipt-reply.ts`.

Status:

**Awaiting successful same-number reconnect + controlled webhook cutover.**

### LDI-002 — Embedded Signup authorization incomplete

**Software diagnostics/recovery fix merged in PR #40; external Meta/browser cause still requires controlled live reproduction.**

The application can now distinguish lifecycle stages and recover safely without guessing the external cause.

Status:

**Next step: controlled Production same-number reconnect after Vercel Production is READY on PR #40 squash SHA.**

### LDI-003 — Manual outbound failure

**Root cause confirmed; PR #39 code fix merged; operational credential transition still pending.**

Production connection still references legacy:

`env:WHATSAPP_ACCESS_TOKEN`

PR #39 prevents false-ready state and impossible send attempts. Operational closure requires successful same-number reconnect so the credential becomes modern `gcp-sm://...` and readiness becomes `ready`.

### LDI-004 — Onboarding response-mode impossible state

**Root cause: CONFIRMED; awaiting PR C.**

No PR C work has started.

---

## Immediate webhook cutover decision

**NO-GO remains in force.**

PR #40 being merged does not authorize webhook cutover.

Before cutover:

1. Vercel Production must be READY on `a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`.
2. Config ending `1674` must be reconfirmed.
3. Existing connection/account scope must be inspected read-only.
4. Owner/admin must perform a controlled reconnect of the SAME receiving number only.
5. Credential reference must transition to modern `gcp-sm://...`.
6. PR #39 readiness must become `ready`.
7. Account scope and historical customers/Contacts/identities/conversations/messages must remain unchanged.
8. Stop on any different-number conflict or ambiguous completion.

Only after this operational transition is verified may a separate webhook cutover plan be reviewed.

---

## Approved repair sequence

### PR A — WhatsApp credential/runtime readiness — MERGED as PR #39

Completed.

### PR B — Embedded Signup Production diagnostics and recovery — MERGED as PR #40

Completed in code and independently reviewed.

### Controlled same-number Production reconnect — NEXT

Purpose:

- move the existing legacy credential reference to modern Secret Manager through the reviewed Embedded Signup flow;
- preserve the same receiving account scope and all historical data;
- prove PR #39 outbound readiness becomes ready.

This is an operational action, not a new feature PR.

### Controlled operational Meta webhook cutover — AFTER RECONNECT VERIFICATION

Only after modern credential readiness is proven.

### PR C — Onboarding response-mode state machine

Still pending and unstarted.

### Contacts PR 2

Resume only after messaging foundation and webhook cutover are production-verified.

---

## Contacts foundation remains intact

Contacts PR 1 remains closed and production-verified:

- `contacts`
- `contact_channel_identities`
- legacy compatibility via `legacy_customer_id`
- Contacts UI remains disabled
- no Contacts PR 2 dual-write has started

The temporary PR1→PR2 gap remains acceptable only while Contacts runtime/UI remains disabled.

---

## لا تفعل الآن

- لا تغيّر Meta webhook callback الآن.
- لا توقف Cloud Run القديم الآن.
- لا تعمل blind retry إذا كانت نتيجة reconnect غامضة.
- لا تربط رقمًا مختلفًا عن الرقم الحالي.
- لا ترسل رسالة اختبار حقيقية قبل authorization منفصل.
- لا تبدأ Contacts PR 2.
- لا تبدأ PR C داخل هذه المرحلة التشغيلية.
- لا تنقل legacy token values يدويًا.
