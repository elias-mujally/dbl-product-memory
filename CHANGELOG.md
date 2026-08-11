# سجل تطور المنتج

> التواريخ أدناه تسجل مراحل المنتج الأساسية، وليست كل commit صغير.

## 2026-08-11

### اعتماد Contacts Foundation تقنيًا

- تم اعتماد `CONTACTS_ARCHITECTURE.md` كخطة WhatsApp-first, Channel-ready.
- تم إجراء Technical Audit كامل على `main` بواسطة Codex دون تعديل أي ملف.
- تم اعتماد `CONTACTS_TECHNICAL_SPEC.md` لـContacts PR 1 — Foundation.
- تمت مراجعة التصميم بشكل مستقل عبر DeepSeek، وتم اعتماد ملاحظاته المتعلقة بالـpreflight، PR1→PR2 gap، وحجم migration الفعلي.

### Production Contacts Preflight — GO

تم تنفيذ mandatory read-only preflight على Production قبل أي Contacts migration.

النتيجة:

- verdict: **GO**
- total customers: **1**
- workspaces with customers: **1**
- customer-linked WhatsApp connections: **1**
- workspace/connection mismatches: **0**
- missing `phone_number_id` account scope: **0**
- canonical identity collision groups: **0**
- WhatsApp ID null/empty/whitespace/length anomalies: **0**
- normalized identity collisions: **0**
- legacy mapping violations: **0**
- missing `profile_name`: **0**
- missing `phone_number_normalized`: **0**
- connection state: `connected` × 1

Migration size was classified as **Tiny**.

Approved consequence:

- simple transactional backfill is appropriate;
- batching is not required;
- online/concurrent index construction is not required;
- no remediation is required before PR 1;
- Contacts PR 1 implementation is now authorized.

The preflight modified no files, schema, production data, migrations, or configuration.

Reference: `CONTACTS_PRODUCTION_PREFLIGHT.md`.

### Contacts PR sequencing confirmed

- PR 1: schema/RLS/backfill/tests only.
- PR 2: immediately follow with WhatsApp ingestion dual-write + idempotent catch-up backfill.
- PR 2 must close the ingestion race before final catch-up validation.
- `/contacts` remains disabled until PR 2 proves complete Contact coverage.
- PR 3: Contacts MVP UI only after PR 2.

## 2026-08-09

### دمج PR #36 — Meta administrator testing readiness + Vercel WIF

- تم تأكيد إضافة نطاق Production `https://dbl-employee-ai.vercel.app/` يدويًا إلى Meta Allowed Domains for the JavaScript SDK.
- بقي Meta Embedded Signup config الصحيح المنتهي بـ`1674` دون تغيير.
- تم تحويل PR #36 من Draft إلى Ready for Review ثم squash merge على الرأس المراجع:
  - `a685d549a6f8c9eb2ab18202456171f56b558c86`
- Squash/main SHA الجديد:
  - `d4d5dbfa988dc2e653f0164193fb9df5a4aba5f2`
- تم الدمج مباشرة عبر GitHub مع حماية expected head SHA.

### PR #36 قبل الدمج

- بدأ PR #36 لفتح Embedded Signup للاختبار الإداري الآمن أثناء انتظار موافقات Meta الخارجية.
- تمت إضافة متغيرات Meta/WhatsApp إلى Vercel Production وPreview كـSensitive variables.
- تم اعتماد `META_EMBEDDED_SIGNUP_RELEASE_STAGE=testing`.
- تم إصلاح sequencing بحيث SDK و`FB.init` وprepared state يجهزون قبل click، و`FB.login` يعمل synchronous داخل user gesture.
- تم اعتماد Vercel OIDC + Google Workload Identity Federation بدل إنشاء JSON key طويل العمر لحساب WhatsApp runtime.

### WIF / IAM

- Workload Identity Pool: `dbl-vercel-runtime`.
- Provider: `vercel-dbl-employee-ai`.
- trust مقيد على فريق DBL ومشروع `dbl-employee-ai` وPreview/Production.
- تم منح `roles/iam.workloadIdentityUser` فقط على:
  - `dbl-employee-ai-runtime@dbl-employee-ai.iam.gserviceaccount.com`
- لم يُمنح project-wide role للهوية الفدرالية.
- لم تُمنح صلاحيات WhatsApp Secret Manager لـ`vertex-express`.
- لم يُنشأ أي JSON key جديد.

### Live Preview verification

- WIF probe: passed.
- Vercel OIDC → Google STS → service-account impersonation: passed.
- Secret Manager authorization probe: passed.
- `metaConfigured=true`.
- Facebook SDK و`FB.init`: passed.
- prepared server state: passed.
- Connect button readiness: passed.
- Meta registration UI ظهرت يدويًا عند الضغط على Connect.
- لم تتم أي WABA/phone mutation.

### الخطوة بعد الدمج

- انتظار Vercel Production deployment حتى READY.
- تنفيذ safe Production smoke verification لمسار `/settings/whatsapp`.
- بعدها الانتقال إلى Contacts MVP ثم Analytics MVP.

## 2026-08-08

### دمج PR #35 وإطلاق approval/idempotency الجديد

- اجتاز الرأس `f09dc02e54a2c3a97805df4400bfc1b81a40826a` المراجعة المستقلة النهائية دون High أو Medium findings.
- تم squash merge لـPR #35.
- Squash/main SHA: `b14c1fe2e144cb58cd0618501abe3f10c5a88494`.
- تم تطبيق migration `20260802172230_knowledge_create_idempotency.sql`.
- Vercel production deployment أصبح READY.
- لم تظهر runtime error clusters أو 5xx أو regressions بعد النشر.

### إغلاق آخر مشكلتين في PR #35

- approval commit مع ضياع الاستجابة أصبح يستخدم authoritative reconciliation.
- create commit مع ضياع الاستجابة ثم reload/remount أصبح محميًا بـsession-scoped idempotency attempt UUID.

التحقق النهائي للرأس المدموج:

- Vitest 431/431.
- Authenticated E2E 12/12.
- Foundation CI passed.
- Supabase reset passed.
- pgTAP passed.
- Production build passed.

## 2026-08-05

- إنشاء مستودع `dbl-product-memory` كذاكرة مؤسسية مستقلة للمشروع.
- توثيق الرؤية، الحالة الحالية، القرارات، المشاكل، والعوائق.

## 2026-08-02

### Knowledge Hub وإعادة تصميم تجربة التعليم

- دمج PR #34.
- تحويل `/knowledge` إلى مركز “ذاكرة الموظف الذكي”.
- تقسيم المعرفة إلى معلومات النشاط، المنتجات والخدمات، السياسات، الأسئلة الشائعة، المستندات، والمعرفة الأخرى.
- إضافة AI Employee Readiness بنسبة 30/30/20/20.
- فصل Documents كـoptional enrichment.
- إصلاح round-trip preservation للحقول المخفية والسياسات والتسعير.

## 2026-08-01

- دمج PR #32: ترجمة protected product surfaces بالعربية والإنجليزية.
- إضافة authenticated Playwright E2E ببيانات اصطناعية محلية.
- دمج PR #33: إضافة Knowledge إلى primary navigation وفصلها عن Knowledge settings.

## 2026-07-29 إلى 2026-07-30

- دمج PR #29: Privacy، Terms، Data Deletion العامة والثنائية اللغة.
- دمج PR #30: localization/theme foundation.
- دمج PR #31: ترجمة AppShell وAuth flows وتحسين accessibility.

## 2026-07-28

- التحقيق العميق في Meta Embedded Signup.
- إثبات أن config القديم يعيد ordinary Facebook Login.
- إنشاء config جديد صحيح ينتهي بـ1674.
- بدء Independent Tech Provider qualification.
- Business Verification بقي pending.

## 2026-07-26

- دمج PR #25: onboarding وAI Settings discoverability.
- دمج PR #26: Meta Embedded Signup foundation، Secret Manager، secure session state، disconnect/reconnect.
- دمج PR #27: فصل WhatsApp credential project عن Vertex project.
- دمج PR #28: تأمين legacy reviews table.

## 2026-07-22 إلى 2026-07-24

- دمج onboarding wizard خمس مراحل.
- إضافة AI Employee customization.
- إضافة review-only mode.
- إضافة safe conversational replies.
- دعم Vertex AI auth على Vercel.

## 2026-07-21

- إضافة automatic grounded WhatsApp AI replies.
- اكتشاف production Supabase schema drift.
- إنشاء forward-only reconciliation migration واختبارها في بيئة معزولة.

## 2026-07-19 إلى 2026-07-20

- إصلاح registration/dashboard errors الناتجة عن schema ناقصة.
- إصلاح Cloud Run callback origin.
- إعادة تصميم SaaS UI.
- تحسين dashboard resilience.
- الانتقال من Gemini API key إلى Vertex AI.

## 2026-07-18

- إصلاح pnpm 11 build approvals وDocker install policy.
- إضافة WhatsApp E2E text receipt flow.
- توثيق الفرق بين webhook verification وWABA subscription.

## 2026-07-16 إلى 2026-07-17

- تأسيس المشروع.
- إضافة Supabase tenant foundation.
- إضافة Auth وWorkspace onboarding.
- إضافة WhatsApp messaging foundation.
- إضافة Knowledge Base foundation.
- إضافة grounded AI drafts.
- إضافة reviewed manual WhatsApp sending.
- إضافة Meta integration readiness foundation.
