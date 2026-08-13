# الحالة الحالية

آخر تحديث: **2026-08-13**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp integration يمر حاليًا بمرحلة إصلاح Production foundation قبل متابعة Contacts PR 2.

تم إغلاق **Contacts PR 1 — Foundation** بالكامل، وتم دمج **PR #38 — WhatsApp Account-Scope Lifecycle Guard**، وتم الآن كذلك دمج **PR #39 — WhatsApp Credential / Runtime Readiness** بعد مراجعة مستقلة نهائية دون High أو Medium findings.

الحالة الحالية:

> **PR #39 merged. Next engineering task: PR B — Embedded Signup Production diagnostics and recovery. Meta webhook cutover remains NO-GO. Contacts PR 2 remains blocked.**

---

## Production الحالي

- Production app: `https://dbl-employee-ai.vercel.app`
- current GitHub `main` after PR #39 merge: `7f5d78ee3441be4cce8c8671992f08cb98b43717`
- PR #39 reviewed head: `e7a3d50b7d2b2328cd6bfc7d2c91231c6efc2d7b`
- PR #39 squash SHA: `7f5d78ee3441be4cce8c8671992f08cb98b43717`
- PR #39: **MERGED via squash**
- Meta webhook routing: **unchanged; still legacy Cloud Run**

Reference:

`events/2026-08-13-pr39-whatsapp-runtime-readiness-merged.md`

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

### Meta webhook ownership

Classification:

`CONFIRMED_LEGACY_CLOUD_RUN`

The active Meta WhatsApp callback is still the old Cloud Run endpoint, not the canonical Vercel webhook.

Cloud Run service:

`dbl-employee-ai-git`

Serving traffic:

- revision `00024`: **100%**
- newer revision built from current main: **0%**

The 100%-serving revision contains the retired static responder:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

Recent read-only Cloud Run logs confirmed five HTTP 200 POST requests to `/api/webhooks/whatsapp` during the manual testing window.

Reference:

`events/2026-08-12-meta-webhook-legacy-cloud-run-confirmed.md`

---

## PR #39 — WhatsApp Credential / Runtime Readiness — MERGED

### الهدف

PR #39 لا ينقل credential القديمة ولا يغير Meta callback. وظيفته أن يجعل runtime الحالي يفهم حقيقة جاهزية outbound بدل اعتبار أي connection بحالة `connected` جاهزة للإرسال تلقائيًا.

### Authoritative readiness

Readiness أصبحت server-side derived وليست persisted state جديدة.

تفرق بين حالات مثل:

- modern Secret Manager ready
- legacy `env:WHATSAPP_ACCESS_TOKEN` requiring same-number reconnect
- missing / invalid / unknown credential reference
- WIF/auth/store/secret failures
- unsupported/invalid provider configuration
- incomplete account scope

### Manual outbound safety

المراجعة المستقلة وجدت Medium واحدًا في النسخة الأولى من PR #39: readiness كانت تُفحص بعد durable reservation، ما يسمح بتكرار failed requests/quota عند reload وrequest ID جديد.

تم الإصلاح بحيث يصبح الترتيب:

```text
authenticated workspace
→ trusted conversation
→ trusted WhatsApp connection
→ early readiness check
→ durable reservation only if eligible
→ late readiness/provider check for TOCTOU
→ send
```

Final re-review أثبت أن محاولتين manual مختلفتين بـrequest IDs مختلفة على نفس legacy connection تنتجان:

- 0 reservation RPC calls
- 0 outbound requests/messages/attempts
- 0 quota consumption
- 0 Meta/provider calls
- reviewed draft preserved

### Automatic outbound

Automatic path بقي deterministic وآمن:

- legacy blocked state = zero Meta call
- no retry loop
- no duplicate request
- generated content preserved

### UX

Connection state وoutbound readiness أصبحا منفصلين.

يمكن أن يبقى الاتصال ظاهرًا كـConnected بينما تظهر للـowner/admin رسالة أن الإرسال يحتاج إعادة ربط **نفس الرقم**.

لا blind retry guidance، ولا ادعاء أن الاتصال disconnected.

### Final review

Reviewed head:

`e7a3d50b7d2b2328cd6bfc7d2c91231c6efc2d7b`

Result:

- High: **0**
- Medium: **0**
- Production risk: **Low**
- Vitest: `481 passed, 5 skipped`
- pgTAP: `466 passed`
- PR #38 T1–T7: passed
- completion service contract: 5 passed
- authenticated E2E: 12 passed
- Vercel Preview: ready

### Merge

- squash SHA: `7f5d78ee3441be4cce8c8671992f08cb98b43717`
- merge method: squash with exact-head protection
- GitHub confirmed successful merge

### مهم

PR #39 **لا يجعل Production outbound يعمل وحده بعد الدمج**. الاتصال الإنتاجي ما زال يستخدم legacy credential ويحتاج owner/admin same-number reconnect ناجح عبر Embedded Signup حتى ينتقل إلى modern Secret Manager credential.

---

## Late-discovered production issues

Reference:

`LATE_DISCOVERED_ISSUES.md`

### LDI-001 — Static WhatsApp acknowledgement

**Root cause: CONFIRMED.**

The old Cloud Run callback/revision is still handling inbound Meta traffic and still contains `receipt-reply.ts`.

Status:

**Awaiting PR B + successful same-number reconnect + controlled webhook cutover.**

### LDI-002 — Embedded Signup authorization incomplete

**Failure stage confirmed; exact external/browser root cause still open.**

Proven path:

```text
SDK ready
→ prepared state ready
→ synchronous FB.login called
→ callback returned without authResponse.code
→ authorization_incomplete
→ no completion mutation
```

Status:

**PR B is now the next engineering task.**

Testing-mode Meta notice should remain truthful but become low-prominence/contextual rather than a permanent alarming banner.

### LDI-003 — Manual outbound failure

**Root cause: CONFIRMED; readiness/UX fix merged in PR #39, operational transition still pending.**

Production connection still references:

`env:WHATSAPP_ACCESS_TOKEN`

PR #39 now prevents false-ready state, blocks impossible manual/automatic sends before Meta, avoids duplicate durable records/quota, and guides owner/admin to same-number reconnect.

The actual credential transition to Secret Manager still requires a successful Embedded Signup reconnect.

### LDI-004 — Onboarding response-mode impossible state

**Root cause: CONFIRMED.**

Persisted `automatic_replies_enabled=true` can coexist with live readiness that makes Automatic ineligible, producing checked+disabled state and `mode_invalid` on submission.

Status:

**High — planned PR C.**

---

## Immediate webhook cutover decision

**NO-GO remains in force.**

Reason:

- split-runtime is proven;
- PR #39 makes outbound readiness truthful and fail-closed;
- but Production still lacks a modern credential until same-number reconnect succeeds;
- Embedded Signup currently returns `authorization_incomplete` in Production;
- changing Meta callback before PR B/reconnect could produce inbound delivery with no outbound capability.

---

## Approved repair sequence

### PR A — WhatsApp credential/runtime readiness — MERGED as PR #39

Completed:

- credential classification
- server-authoritative readiness
- privacy-safe provider diagnostics
- manual/automatic pre-network blocking for legacy state
- pre-reservation manual gate
- late TOCTOU boundary retained
- settings/conversation reconnect guidance
- no Meta webhook change

### PR B — Embedded Signup Production diagnostics and recovery — NEXT

Goals:

- safe SDK/login callback telemetry;
- reproduce Production callback-without-code safely;
- make same-number reconnect reliably executable;
- contextualize testing-stage Meta notice;
- no provider asset mutation in automated tests.

### Controlled operational webhook cutover

Only after PR B + successful modern credential transition:

- verify Vercel GET challenge and POST signature handling;
- confirm modern credential is readable;
- prove manual outbound;
- prove intended automatic/review-only behavior;
- change Meta callback in an approved operational window;
- confirm signed inbound reaches Vercel;
- preserve old Cloud Run revision temporarily for rollback;
- retire legacy responder only after Vercel path is verified.

### PR C — Onboarding response-mode state machine

Goals:

- recoverable controlled response-mode selection;
- no checked+disabled invalid state;
- shared readiness authority between UI and activation;
- preserve selection across errors/reload;
- safe employee-test outcome categories.

### Contacts PR 2

Resume only after messaging foundation is healthy and the operational cutover is production-verified.

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
- لا تعمل blind retry للإرسال قبل successful same-number reconnect.
- لا تبدأ Contacts PR 2.
- لا تفتح Contacts UI.
- لا تنقل legacy token values يدويًا.
- لا تبدأ PR C داخل PR B.
