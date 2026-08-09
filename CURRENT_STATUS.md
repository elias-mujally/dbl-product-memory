# الحالة الحالية

آخر تحديث: **2026-08-09**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp/Meta Embedded Signup أصبح جاهزًا تقنيًا للاختبار الإداري بعد دمج PR #36، بينما تبقى موافقات Meta التجارية الخارجية قيد الانتظار.

## الإنتاج الحالي

- Production: `https://dbl-employee-ai.vercel.app`
- `main`: `d4d5dbfa988dc2e653f0164193fb9df5a4aba5f2`
- آخر PR مدمج: **#36**
- PR #35 أطلق approval workflow الداخلي وidempotent Knowledge creation.
- PR #36 أطلق Meta administrator testing readiness وVercel OIDC + Google WIF لمسار WhatsApp Secret Manager.

## PR #36 تم دمجه

PR: **#36 — feat: prepare Meta administrator signup testing**

- الحالة: **MERGED via squash**
- الرأس الذي تمت مراجعته: `a685d549a6f8c9eb2ab18202456171f56b558c86`
- Squash/main SHA: `d4d5dbfa988dc2e653f0164193fb9df5a4aba5f2`
- قبل الدمج تم تأكيد أن Preview وCI خضراء على الرأس المراجع.
- نطاق Production `https://dbl-employee-ai.vercel.app/` أضيف يدويًا إلى Meta Allowed Domains for the JavaScript SDK قبل الدمج.
- Meta config الصحيح المنتهي بـ`1674` بقي دون تغيير.

### ما الذي أصبح موجودًا؟

- technical runtime readiness منفصلة عن Meta commercial approval.
- `metaConfigured` يعتمد على إعدادات runtime الفعلية.
- testing/pending Meta approval banner.
- Facebook SDK و`FB.init` يكتملان قبل تفعيل Connect.
- prepared server state قصير العمر يُنشأ قبل click.
- `FB.login` يعمل synchronously داخل user gesture دون `await` قبله.
- owner/admin فقط يمكنهما بدء الربط.
- agent/viewer لا يملكان أدوات الربط.

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

ومتغيرات WIF:

- `GCP_PROJECT_NUMBER`
- `GCP_SERVICE_ACCOUNT_EMAIL`
- `GCP_WORKLOAD_IDENTITY_POOL_ID`
- `GCP_WORKLOAD_IDENTITY_POOL_PROVIDER_ID`

## Google Cloud / Secret Manager / WIF

المسار المعتمد:

`Vercel OIDC → Google STS → service-account impersonation → Secret Manager`

- Vercel OIDC: Team issuer mode.
- Workload Identity Pool: `dbl-vercel-runtime`
- Provider: `vercel-dbl-employee-ai`
- trust مقيد على فريق DBL ومشروع `dbl-employee-ai` وPreview/Production فقط.
- runtime service account:
  - `dbl-employee-ai-runtime@dbl-employee-ai.iam.gserviceaccount.com`
- federated principal مُنح فقط `roles/iam.workloadIdentityUser`.
- لم يُنشأ JSON key جديد.
- لم تُمنح صلاحيات WhatsApp Secret Manager لـ`vertex-express`.
- JSON credential القديم بقي معزولًا لمسار Vertex AI.

## Live verification قبل الدمج

- WIF probe: Pass.
- Google STS exchange: Pass.
- service-account impersonation: Pass.
- Secret Manager authorization probe: Pass.
- `metaConfigured=true` على Preview.
- Facebook SDK + `FB.init`: Pass.
- prepared state: Pass.
- Connect button readiness: Pass.
- popup/registration page من Meta ظهرت يدويًا عند الضغط على Connect.
- لم تُعدّل WABA أو phone assets أثناء الاختبار.

## Meta Business status

- Business Verification: ما زالت Pending/In progress.
- Independent Tech Provider qualification بدأ لكنه متوقف على Business Verification.
- Advanced Access وApp Review لم يكتملوا بعد.
- external customer onboarding لا يُعتبر production-ready قبل موافقات Meta.

## الخطوة التالية حرفيًا

1. انتظار Vercel Production deployment الناتج عن دمج PR #36 حتى READY.
2. التحقق من `/settings/whatsapp` بحساب owner/admin على Production.
3. التأكد أن:
   - `metaConfigured=true`
   - testing banner ظاهر
   - Connect يعمل
   - Meta popup يفتح على Production
   - لا WIF/STS/Secret Manager errors أو 5xx
4. التوقف قبل أي WABA/phone asset mutation غير مطلوبة.
5. بعد إغلاق تحقق Production، الانتقال إلى **Contacts MVP** ثم **Analytics MVP** كـPRs منفصلة.

## لا تفعل الآن

- لا تنشئ JSON key جديد لـ`dbl-employee-ai-runtime`.
- لا تمنح `vertex-express` صلاحيات WhatsApp Secret Manager.
- لا تغيّر Meta config أو secrets دون blocker مثبت.
- لا تعتبر Business Verification مكتملة حتى تظهر فعليًا في Meta.
- لا تبدأ Contacts أو Analytics قبل إغلاق تحقق Production الخاص بـPR #36.