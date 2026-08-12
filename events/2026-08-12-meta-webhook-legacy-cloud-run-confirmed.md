# 2026-08-12 — Legacy Cloud Run webhook ownership confirmed

## Result

Read-only Production confirmation proved that Meta WhatsApp inbound webhook traffic is still routed to the legacy Cloud Run runtime, not the canonical Vercel webhook.

Classification:

`CONFIRMED_LEGACY_CLOUD_RUN`

No code, production data, Meta configuration, webhook subscription, Cloud Run traffic, Vercel configuration, credentials, or WhatsApp assets were modified during this confirmation.

## Canonical current application

- GitHub `main`: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- Vercel deployment: `dpl_5X9Jtqsb5qYkneNTsL4Dr4ZZVcgB`
- Vercel status: `READY`
- Canonical webhook route in current source:
  - `https://dbl-employee-ai.vercel.app/api/webhooks/whatsapp`

The retired static acknowledgement is absent from current `main`.

## Historical static responder

Historical module:

`server/messaging/whatsapp/receipt-reply.ts`

Introduced in historical commit `fabbd3c` / main squash equivalent `6a899b0`.

Removed in historical commit `44b744e` / main squash equivalent `4ee0a56`.

Known static reply:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

Behavior:

- executed after a newly inserted inbound text message;
- sent directly through the Meta provider;
- bypassed the modern durable outbound request/attempt path;
- did not call AI, retrieval, grounding, Gemini, or Vertex AI;
- duplicate inbound events did not resend it when ingestion returned `inserted=false`.

Conclusion: the static acknowledgement itself consumes zero AI tokens.

## Meta webhook target

The authenticated current DBL Meta app was inspected read-only.

The active WhatsApp webhook callback is classified as the old Cloud Run target:

`dbl-employee-ai-git-….us-central1.run.app/api/webhooks/whatsapp`

It is not the canonical Vercel domain.

The current Embedded Signup configuration remains the named DBL configuration ending in `1674`. The older configuration ending in `3161` is not treated as current.

Visible WhatsApp Business Account subscribed fields included `messages` and other configured v25.0 fields. No subscriptions were changed.

## Cloud Run proof

Cloud Run service:

`dbl-employee-ai-git`

Region:

`us-central1`

Traffic allocation:

- revision `dbl-employee-ai-git-00024-zcv`: **100%**
- newer revision `00057`, built from current main: **0%**
- other inspected revisions: **0%**

The 100%-serving revision was built from source commit:

`d0ba714aed7876f40830939cb883372c24d9da16`

That source contains `receipt-reply.ts` and the retired static acknowledgement.

Recent read-only logs confirmed 5 POST requests to `/api/webhooks/whatsapp` on the serving legacy revision during the manual-production-test window, all HTTP 200. Only request metadata/timestamps were inspected; no payloads or identifiers were opened.

The timestamps correlated with the previously observed Supabase inbound webhook/message activity.

## Split-runtime is proven

Current runtime ownership is split:

```text
Meta inbound webhook
→ legacy Cloud Run revision 00024
→ historical static receipt responder

Current DBL UI / server actions
→ Vercel current main
→ modern outbound / AI / Embedded Signup code
```

This explains why current GitHub/Vercel code did not contain the visible static reply while real inbound WhatsApp traffic continued to receive it.

## Related late-discovered issues

### LDI-001

Root cause is now confirmed:

- Meta callback targets legacy Cloud Run;
- Cloud Run sends traffic to revision 00024 at 100%;
- that revision contains the retired static responder;
- recent Meta-originated requests reached that revision.

Status: **Confirmed root cause — awaiting repair and controlled cutover.**

### LDI-003

Prior audit already proved the current database connection still references legacy credential location:

`env:WHATSAPP_ACCESS_TOKEN`

The current Vercel outbound runtime expects the modern credential path through Secret Manager/WIF. This is why manual outbound currently fails before a Meta Graph API request.

Status: **Confirmed root cause — awaiting PR A.**

### LDI-002

Embedded Signup still requires dedicated Production diagnostics/recovery work. The exact Meta/browser reason for callback-without-code remains unproven.

Status: **Open — PR B planned after PR A.**

### LDI-004

The onboarding response-mode state-machine defect is independently confirmed and requires a separate PR C.

## Immediate cutover decision

Immediate webhook cutover is **NO-GO**.

Risk score from the read-only audit: **8/10 (High)**.

Reason:

- Vercel inbound persistence/signature path is expected to work;
- the obsolete static reply would disappear;
- but manual and automatic outbound delivery are not currently reliable because the active connection still uses the legacy credential reference;
- cutting over now could result in messages being received and stored correctly while customers receive no response.

## Approved sequencing

1. **PR A — WhatsApp credential/runtime readiness**
   - distinguish legacy vs modern credential readiness;
   - add safe provider-construction diagnostics;
   - avoid presenting unusable legacy connection as outbound-ready;
   - prepare same-number credential transition;
   - prove manual/automatic outbound readiness.

2. **PR B — Embedded Signup production diagnostics and recovery**
   - safe browser/SDK callback diagnostics;
   - controlled Production-domain reproduction;
   - make same-number reconnect reliably executable;
   - improve testing-stage notice without misrepresenting Meta approval status.

3. **Controlled operational cutover**
   - verify canonical Vercel GET challenge/signature path;
   - verify Secret Manager credential readiness and manual outbound;
   - change Meta callback during an approved window;
   - confirm signed inbound delivery reaches Vercel;
   - preserve old callback/revision for rollback initially;
   - retire legacy responder only after new path is proven.

4. **PR C — Onboarding response-mode state machine**
   - recoverable controlled response-mode selection;
   - shared readiness authority;
   - safe test-result categories.

5. Resume **Contacts PR 2** only after the production messaging foundation is healthy.

## Do not do yet

- Do not change Meta webhook routing now.
- Do not shut down or redirect Cloud Run yet.
- Do not begin Contacts PR 2.
- Do not copy legacy token values into application code.
- Do not attempt blind outbound retries while the legacy credential state remains.

## Next engineering task

**PR A — WhatsApp credential/runtime readiness.**
