# سجل تطور المنتج

> التواريخ أدناه تسجل مراحل المنتج الأساسية، وليست كل commit صغير.

## 2026-08-08

### استئناف PR #35 بعد عودة Codex

- عاد Codex بعد توقف الحصة السابقة.
- تم دفع آخر إصلاح محلي كان عالقًا في:
  - `components/knowledge-task-form-actions.tsx`
  - `tests/e2e/knowledge-idempotency.spec.ts`
- تم تثبيت السلوك الصحيح لـ“Save and add another”:
  - عملية جديدة فعلًا تحصل على idempotency key جديد.
  - retry أو النقر المكرر لنفس العملية يحتفظ بالمفتاح نفسه.
- الرأس الجديد لـPR #35 أصبح:
  - `c0421c86bcd2ca87c0ff338a3de77256d061d0e1`
- التحقق النهائي:
  - focused tests 13/13
  - Vitest 423/423
  - Foundation CI passed
  - Supabase reset passed
  - pgTAP passed
  - Authenticated E2E passed
  - Vercel Preview Ready
- المراجعة الذاتية الحالية:
  - High: none
  - Medium: none
  - readiness: 99/100
- PR #35 ما زال Draft وغير مدمج بانتظار مراجعة مستقلة نهائية ثم squash merge إذا بقي نظيفًا.

### Meta / Google Cloud

- Business Verification لدى Meta ما زالت Pending/In progress لأكثر من أسبوع.
- تم التأكيد أن Google Cloud وSecret Manager وحساب الخدمة والصلاحيات اللازمة لتخزين أسرار WhatsApp قد جُهزت سابقًا بالكامل.
- `GOOGLE_SERVICE_ACCOUNT_JSON` موجود على Vercel.
- لا حاجة لإعادة إنشاء بنية Google Cloud؛ المتبقي فقط تدقيق متغيرات Meta/WhatsApp في Vercel عند الحاجة.

## 2026-08-05

- إنشاء مستودع `dbl-product-memory` كذاكرة مؤسسية مستقلة للمشروع.
- توثيق الرؤية، الحالة الحالية، القرارات، المشاكل، والعوائق.
- PR #35 ما زال Draft بانتظار دفع آخر إصلاح محلي بعد تجدد قدرة Codex الخارجية.

## 2026-08-02

### Knowledge Hub وإعادة تصميم تجربة التعليم

- دمج PR #34.
- تحويل `/knowledge` إلى مركز “ذاكرة الموظف الذكي”.
- تقسيم المعرفة إلى:
  - معلومات النشاط
  - المنتجات والخدمات
  - السياسات
  - الأسئلة الشائعة
  - المستندات
  - المعرفة الأخرى
- إضافة forms متخصصة بدل نموذج تقني واحد.
- إخفاء Source Type وPriority والمفاهيم الداخلية.
- إضافة AI Employee Readiness:
  - business 30%
  - products/services 30%
  - policies 20%
  - FAQs 20%
- فصل Documents كـoptional enrichment.
- إصلاح round-trip preservation للحقول المخفية والسياسات والتسعير.

### PR #35

- بدء تصحيح approval UX.
- توضيح أن review داخل Workspace العميل.
- إضافة owner/admin publish، agent submit، viewer read-only.
- إضافة review queue.
- اكتشاف create-and-publish duplication risk.
- الموافقة على migration آمنة لـidempotency.

## 2026-08-01

- دمج PR #32: ترجمة protected product surfaces بالعربية والإنجليزية.
- إضافة authenticated Playwright E2E ببيانات اصطناعية محلية.
- إصلاح bidi/date/validation/accessibility في Conversations/Knowledge/Settings/Onboarding/WhatsApp.
- دمج PR #33: إضافة Knowledge إلى primary navigation وفصلها عن Knowledge settings.

## 2026-07-29 إلى 2026-07-30

- دمج PR #29: Privacy، Terms، Data Deletion العامة والثنائية اللغة.
- تصحيح legal overclaims وdeletion process وcanonical domain.
- دمج PR #30: localization/theme foundation.
- فصل UI locale عن AI reply language.
- إضافة locale provenance وSSR lang/dir/theme.
- دمج PR #31: ترجمة AppShell وAuth flows وتحسين accessibility.

## 2026-07-28

- التحقيق العميق في Meta Embedded Signup.
- بناء عدة isolated revisions وprivacy-safe debug panels.
- إثبات أن config القديم يعيد ordinary Facebook Login.
- إنشاء standalone minimal reproduction.
- إنشاء config جديد صحيح ينتهي بـ1674.
- بدء Independent Tech Provider qualification.
- Business Verification بقي pending.

## 2026-07-26

- دمج PR #25: جعل onboarding وAI Settings قابلة للاكتشاف.
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
