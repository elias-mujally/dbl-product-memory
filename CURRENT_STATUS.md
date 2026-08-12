# الحالة الحالية

آخر تحديث: **2026-08-12**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp integration يمر حاليًا بمرحلة إصلاح Production foundation قبل متابعة Contacts PR 2.

تم إغلاق **Contacts PR 1 — Foundation** بالكامل، وتم دمج **PR #38 — WhatsApp Account-Scope Lifecycle Guard** بنجاح.

بعد الاختبار اليدوي على Production تم اكتشاف مشاكل متأخرة، ثم أثبتت مراجعة read-only أن WhatsApp runtime الحالي منقسم فعليًا بين Cloud Run قديم وVercel الحالي.

الحالة الحالية:

> **Contacts PR 2 is temporarily blocked. Next engineering task: PR A — WhatsApp credential/runtime readiness. Immediate Meta webhook cutover is NO-GO.**

---

## Production الحالي

- Production app: `https://dbl-employee-ai.vercel.app`
- current GitHub `main`: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- current Vercel deployment: `dpl_5X9Jtqsb5qYkneNTsL4Dr4ZZVcgB`
- Vercel status: **READY**
- PR #38: **MERGED via squash**

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

## Late-discovered production issues

Reference:

`LATE_DISCOVERED_ISSUES.md`

### LDI-001 — Static WhatsApp acknowledgement

**Root cause: CONFIRMED.**

The old Cloud Run callback/revision is still handling inbound Meta traffic and still contains `receipt-reply.ts`.

Important correction:

The static acknowledgement itself does **not** invoke AI and consumes **0 AI tokens**. Any later AI run is a separate operation.

Status:

**Awaiting PR A + PR B + controlled webhook cutover.**

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

**Planned PR B after PR A.**

Testing-mode Meta notice should remain truthful but become low-prominence/contextual rather than a permanent alarming banner.

### LDI-003 — Manual outbound failure

**Root cause: CONFIRMED.**

Active Production connection still references legacy credential location:

`env:WHATSAPP_ACCESS_TOKEN`

Current Vercel outbound runtime expects the modern Secret Manager/WIF path.

The manual-send path fails during provider construction before Meta Graph API request with safe category:

`provider_configuration_missing`

Status:

**High — next repair target in PR A.**

### LDI-004 — Onboarding response-mode impossible state

**Root cause: CONFIRMED.**

Persisted `automatic_replies_enabled=true` can coexist with live readiness that makes Automatic ineligible. The form then renders Automatic checked + disabled; disabled controls are omitted from form submission, causing `mode_invalid` on activation.

Status:

**High — planned PR C.**

---

## Immediate webhook cutover decision

**NO-GO.**

Risk score: **8/10 High**.

Reason:

- Vercel inbound verification/persistence is expected to work;
- moving Meta callback now would remove the obsolete static acknowledgement;
- however the current Vercel outbound path still cannot use the legacy credential reference;
- customers could therefore have inbound messages stored successfully but receive no reply.

Do not change Meta callback until PR A and PR B are complete and outbound readiness is proven.

---

## Approved repair sequence

### PR A — WhatsApp credential/runtime readiness — NEXT

Goals:

- distinguish legacy credential state from modern Secret Manager readiness;
- add privacy-safe provider-construction diagnostics;
- stop representing an unusable legacy connection as fully outbound-ready;
- prepare the safe same-number credential transition;
- verify manual and automatic outbound readiness;
- no Meta webhook cutover inside this PR.

### PR B — Embedded Signup Production diagnostics and recovery

Goals:

- safe SDK/login callback telemetry;
- reproduce Production callback-without-code safely;
- make same-number reconnect reliably executable;
- contextualize testing-stage Meta notice;
- no provider asset mutation in automated tests.

### Controlled operational webhook cutover

Only after PR A + PR B:

- verify Vercel GET challenge and POST signature handling;
- confirm modern credential is readable;
- prove manual outbound;
- change Meta callback in an approved operational window;
- confirm signed inbound reaches Vercel;
- preserve old Cloud Run callback/revision temporarily for rollback;
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
- لا تعمل blind retry للإرسال اليدوي قبل إصلاح credential readiness.
- لا تبدأ Contacts PR 2.
- لا تفتح Contacts UI.
- لا تنقل legacy token values يدويًا إلى التطبيق.
- لا تعتبر split-runtime مجرد hypothesis بعد الآن؛ أصبح مثبتًا.
