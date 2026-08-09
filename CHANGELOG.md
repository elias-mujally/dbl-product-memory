# سجل تطور المنتج

> التواريخ أدناه تسجل مراحل المنتج الأساسية، وليست كل commit صغير.

## 2026-08-09

### PR #36 — Meta administrator testing readiness + Vercel WIF

- بدأ PR #36 لفتح Embedded Signup للاختبار الإداري الآمن أثناء انتظار موافقات Meta الخارجية.
- تم تدقيق Vercel وتبيّن أن متغيرات Meta/WhatsApp كانت مفقودة؛ تمت إضافتها إلى Production وPreview كـSensitive variables.
- تم تأكيد استخدام Meta config الصحيح المنتهي بـ`1674` وعدم استخدام القديم `3161`.
- تم الحفاظ على `META_EMBEDDED_SIGNUP_RELEASE_STAGE=testing` وعدم الادعاء بجاهزية onboarding للعملاء الخارجيين.
- تم إصلاح sequencing:
  - SDK و`FB.init` قبل تفعيل الزر.
  - prepared state قبل click.
  - `FB.login` synchronous داخل user gesture دون `await` قبله.
- اكتُشف أن حساب runtime المخصص لـWhatsApp لا يملك JSON key، وأن JSON الموجود في Vercel مرتبط بمسار Vertex AI ولا يملك Secret Manager role المطلوبة.
- بدل توسيع صلاحيات `vertex-express` أو إنشاء key طويل العمر، تم اعتماد Vercel OIDC + Google Workload Identity Federation.

### WIF / IAM

- Vercel OIDC: Team issuer mode.
- Workload Identity Pool: `dbl-vercel-runtime`.
- Provider: `vercel-dbl-employee-ai`.
- trust مقيد على فريق DBL ومشروع `dbl-employee-ai` وPreview/Production فقط.
- تم منح `roles/iam.workloadIdentityUser` فقط للهوية الفدرالية على:
  - `dbl-employee-ai-runtime@dbl-employee-ai.iam.gserviceaccount.com`
- لم يُمنح project-wide role للهوية الفدرالية.
- لم تُمنح صلاحيات WhatsApp Secret Manager لـ`vertex-express`.
- لم يُنشأ أي JSON key جديد.
- Secret Manager custom role بقي بأربع الصلاحيات المحدودة السابقة.

### Vercel WIF environment

أضيفت إلى Production وPreview:

- `GCP_PROJECT_NUMBER`
- `GCP_SERVICE_ACCOUNT_EMAIL`
- `GCP_WORKLOAD_IDENTITY_POOL_ID`
- `GCP_WORKLOAD_IDENTITY_POOL_PROVIDER_ID`

### Live Preview verification

- WIF probe: passed.
- Vercel OIDC → Google STS → service-account impersonation: passed.
- Secret Manager authorization readiness endpoint: HTTP 200 `ready`.
- لم تتم قراءة customer secret أو secret value.
- `metaConfigured=true`.
- Facebook SDK loaded.
- `FB.init` passed.
- prepared server state passed.
- Connect button enabled only after readiness.
- لا runtime errors أو 5xx مرتبطة بالمسار.
- تم إصلاح الفصل بين OIDC token audience وSTS audience حسب متطلبات Google.
- Controlled click وصل إلى Facebook SDK وطلب Facebook، لكن automation surface لم يعرض popup المرئي، لذلك النتيجة المرئية داخل Meta تحتاج تأكيدًا يدويًا لاحقًا.
- لم تتم أي WABA/phone mutation.

الرأس الحالي لـPR #36:

- `a685d549a6f8c9eb2ab18202456171f56b558c86`

الحالة:

- Draft.
- غير مدمج.
- infrastructure blocker تم حله.
- الخطوة التالية: independent technical review ثم merge إذا لم تظهر High/Medium findings.

## 2026-08-08

### دمج PR #35 وإطلاق approval/idempotency الجديد

- اجتاز الرأس `f09dc02e54a2c3a97805df4400bfc1b81a40826a` المراجعة المستقلة النهائية دون High أو Medium findings.
- تم squash merge لـPR #35.
- Squash/main SHA: `b14c1fe2e144cb58cd0618501abe3f10c5a88494`.
- تم تطبيق migration:
  - `20260802172230_knowledge_create_idempotency.sql`
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
