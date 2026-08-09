# الحالة الحالية

آخر تحديث: **2026-08-09**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp/Meta Embedded Signup قيد إنهاء جاهزية الاختبار الإداري بينما تبقى موافقات Meta التجارية الخارجية قيد الانتظار.

## الإنتاج الحالي

- Production: `https://dbl-employee-ai.vercel.app`
- `main`: `b14c1fe2e144cb58cd0618501abe3f10c5a88494`
- آخر PR مدمج: **#35**
- PR #35 أطلق approval workflow الداخلي وidempotent Knowledge creation بأمان.
- لا توجد runtime error clusters أو 5xx مرتبطة بإصدار PR #35.

## PR الحالي

PR المفتوح: **#36 — Meta administrator testing readiness**

- الحالة: **Draft، غير مدمج**.
- الفرع: `codex/meta-admin-testing-readiness`
- الرأس الحالي: `a685d549a6f8c9eb2ab18202456171f56b558c86`
- Preview deployment: `HP6Nh1WjbLPRHLHfQ7QECUL2sAWR`
- Preview status: **READY**
- Production لم يُنشر من PR #36 ولم يتحول traffic إليه.

### ما الذي أصلحه PR #36؟

- فصل technical runtime readiness عن Meta commercial approval.
- `metaConfigured` يعتمد على إعدادات runtime الفعلية فقط.
- إضافة testing/pending Meta approval banner.
- إصلاح sequencing لـFacebook SDK وEmbedded Signup:
  - SDK و`FB.init` يكتملان أولًا.
  - prepared server state يُنشأ قبل تفعيل الزر.
  - `FB.login` يعمل synchronously داخل user gesture دون `await` قبله.
- owner/admin فقط يمكنهما بدء الربط.
- agent/viewer لا يملكان أدوات الربط.
- لا تغيير في WABA/phone assets ضمن CI أو الاختبارات الآلية.

## Vercel / Meta environment

تمت إضافة متغيرات Meta/WhatsApp المطلوبة إلى Production وPreview كـSensitive variables:

- `META_APP_ID`
- `META_APP_SECRET`
- `META_EMBEDDED_SIGNUP_CONFIG_ID`
- `WHATSAPP_CREDENTIAL_STORE`
- `WHATSAPP_SECRET_MANAGER_PROJECT`
- `WHATSAPP_PROVIDER`
- `WHATSAPP_GRAPH_API_VERSION`
- `WHATSAPP_VERIFY_TOKEN`
- `META_EMBEDDED_SIGNUP_RELEASE_STAGE=testing`
- `GOOGLE_SERVICE_ACCOUNT_JSON` كان موجودًا مسبقًا.

تم التأكد من استخدام Meta Embedded Signup config الصحيح المنتهي بـ`1674`، وعدم استخدام config القديم المنتهي بـ`3161`.

## Google Cloud / Secret Manager / WIF

تم اعتماد بنية قصيرة العمر بدل إنشاء JSON key جديد لحساب WhatsApp runtime:

`Vercel OIDC → Google STS → service-account impersonation → Secret Manager`

### البنية الحالية

- Vercel OIDC: enabled، Team issuer mode.
- Workload Identity Pool: `dbl-vercel-runtime`
- Provider: `vercel-dbl-employee-ai`
- issuer مقيد على فريق DBL في Vercel.
- claims mapped للفريق والمشروع والبيئة.
- الشرط يسمح فقط بمشروع DBL المحدد وبيئات Preview/Production.
- runtime service account:
  - `dbl-employee-ai-runtime@dbl-employee-ai.iam.gserviceaccount.com`
- federated principal مُنح فقط:
  - `roles/iam.workloadIdentityUser`
- لم يُمنح project-wide role للهوية الفدرالية.
- Secret Manager custom role بقي بأربع الصلاحيات المحددة سابقًا.
- لم تُضف أي صلاحية إلى `vertex-express`.
- لم يُنشأ أي JSON key جديد.
- JSON credential القديم بقي معزولًا لمسار Vertex AI الحالي، وليس WhatsApp Secret Manager.

متغيرات WIF المضافة إلى Vercel Production وPreview:

- `GCP_PROJECT_NUMBER`
- `GCP_SERVICE_ACCOUNT_EMAIL`
- `GCP_WORKLOAD_IDENTITY_POOL_ID`
- `GCP_WORKLOAD_IDENTITY_POOL_PROVIDER_ID`

## Live Preview verification

تم التحقق حيًا من Preview بحساب DBL owner/admin:

- WIF probe: **Pass**
- Vercel OIDC → Google STS → service-account impersonation: **Pass**
- Secret Manager authorization probe: **Pass**
- readiness endpoint: HTTP 200 `ready`
- لم تُقرأ أي secret value أو customer secret.
- `metaConfigured=true`
- Facebook SDK: **Pass**
- `FB.init`: **Pass**
- prepared server state: **Pass**
- Connect button readiness: **Pass**
- لا runtime errors أو 5xx مرتبطة بـWIF/STS/Secret Manager/SDK/prepared-state.

تم إصلاح audience separation:

- OIDC token audience يستخدم صيغة Google resource URL المناسبة.
- STS audience يستخدم الصيغة المطلوبة لـSTS.

### Controlled popup

تم تنفيذ click واحد مصرح به واستدعاء Facebook SDK وطلب Facebook بنجاح، لكن automation surface لم تعرض نافذة popup نفسها؛ لذلك الواجهة المرئية النهائية داخل Meta والنتيجة النهائية ما زالت **غير مؤكدة يدويًا**.

لم تتم محاولة ثانية ولم تُعدّل WABA أو phone assets.

## التحقق الحالي لـPR #36

- Vitest: 458/458 passed قبل آخر runtime verification، مع CI الأخضر على الرأس الحالي.
- Lint: passed.
- Typecheck: passed.
- Production build: passed.
- `git diff --check`: passed.
- Foundation CI: passed.
- Supabase reset + pgTAP: passed.
- Authenticated synthetic E2E: passed/كان آخر تحقق في طور الاكتمال ثم بقي شرط المراجعة النهائية.
- Vercel Preview: READY.
- Secret/PII scan: clean.

## Meta Business status

- Business Verification: ما زالت Pending/In progress.
- Independent Tech Provider qualification بدأ لكنه متوقف على Business Verification.
- Advanced Access وApp Review لم يكتملوا بعد.
- external customer onboarding لا يجب اعتباره production-ready قبل موافقات Meta.

## الخطوة التالية حرفيًا

1. تنفيذ **مراجعة مستقلة نهائية لـPR #36** على الرأس:
   `a685d549a6f8c9eb2ab18202456171f56b558c86`
2. مراجعة خاصة لـ:
   - OIDC/WIF trust restrictions.
   - STS audience handling.
   - service-account impersonation.
   - Secret Manager fail-closed behavior.
   - Meta SDK sequencing.
   - prepared-state lifecycle.
   - testing banner والصلاحيات.
3. إذا لم تظهر High/Medium findings، يمكن squash merge لـPR #36.
4. بعد الدمج: Production deploy + safe runtime verification.
5. يبقى manual confirmation لواجهة Meta popup خطوة يدوية آمنة، دون إكمال asset mutation إلا بموافقة صريحة.
6. بعد إغلاق PR #36 ننتقل إلى **Contacts MVP** ثم **Analytics MVP** كـPRs منفصلة.

## لا تفعل الآن

- لا تنشئ JSON key جديد لـ`dbl-employee-ai-runtime`.
- لا تمنح `vertex-express` صلاحيات WhatsApp Secret Manager.
- لا تغيّر Meta config أو secrets دون blocker مثبت.
- لا تعتبر Business Verification مكتملة حتى تظهر فعليًا في Meta.
- لا تبدأ Contacts أو Analytics قبل إنهاء PR #36 ومراجعته.