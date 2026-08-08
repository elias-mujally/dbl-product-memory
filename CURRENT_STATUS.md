# الحالة الحالية

آخر تحديث: **2026-08-08**

## ملخص سريع

DBL Employee AI أصبح تطبيق SaaS فعلي متعدد المستأجرين، بواجهة عربية/إنجليزية، ونظام معرفة منظم، وذكاء اصطناعي مؤسس على معرفة معتمدة، وتكامل WhatsApp جاهز برمجيًا إلى حد كبير بينما تبقى بعض متطلبات Meta الخارجية قيد الانتظار.

المنتج يملك بنية حقيقية تشمل:

- Authentication وWorkspace onboarding.
- RLS وعزل المستأجرين.
- WhatsApp webhook ingestion.
- Conversations وContacts foundation.
- Knowledge Base ومراجعة واعتماد للمحتوى.
- Grounded AI drafts وautomatic replies.
- AI employee customization.
- Meta Embedded Signup infrastructure.
- صفحات قانونية عامة.
- localization وtheme foundation.
- Knowledge Hub مبني حول تعليم الموظف الذكي.

## أين توقفنا بالضبط؟

### PR #35 تم دمجه وإطلاقه

PR: **#35 — fix: clarify internal knowledge approval workflow**

- الحالة: **MERGED via squash**.
- الرأس الذي اجتاز المراجعة النهائية: `f09dc02e54a2c3a97805df4400bfc1b81a40826a`
- Squash/main SHA: `b14c1fe2e144cb58cd0618501abe3f10c5a88494`
- Vercel deployment: `dpl_3ffgPf2G7fFr6Jb92TcU2wo7R7Q8`
- Production status: **READY**
- Production URL: `https://dbl-employee-ai.vercel.app`
- لا توجد regressions أو runtime error clusters أو 5xx مرتبطة بالإصدار الجديد.

PR #35 أضاف وثبّت:

- توضيح أن المراجعة داخل Workspace العميل وليست بواسطة DBL.
- Owner/Admin: حفظ مسودة أو إتاحة للموظف الذكي.
- Agent: حفظ مسودة أو إرسال للمالك/المدير للمراجعة.
- Viewer: قراءة فقط.
- Review queue للمالك والمدير.
- create idempotency دائم على مستوى قاعدة البيانات.
- pending-state UX لمنع النقرات المتكررة.
- partial-failure recovery إذا نجح الحفظ وفشل الاعتماد.
- reconciliation لحالات approval commit + lost response.
- session-scoped create-attempt UUID يمنع duplicates عبر reload/remount/retry.
- role-boundary coverage واختبارات E2E.

### idempotency الإنتاجية

الهجرة التي تم تطبيقها إنتاجيًا:

- `20260802172230_knowledge_create_idempotency.sql`
- جدول داخلي: `knowledge_source_create_requests`
- unique constraint: `(workspace_id, actor_user_id, idempotency_key)`
- RPC جديد: `create_idempotent_knowledge_source(...)`
- RLS + FORCE RLS.
- `SECURITY DEFINER` مع `search_path=''`.
- request fingerprint باستخدام SHA-256 للمدخلات الدلالية المطبعة.
- الحجز الذري + unique constraint + `SELECT ... FOR UPDATE` للتزامن.
- retry لنفس العملية يعيد نفس source ID بدل إنشاء سجل جديد.

تم التحقق بعد الدمج من أن:

- migration history يحتوي صفًا واحدًا فقط للهجرة.
- الجدول وRPC موجودان.
- RLS وFORCE RLS مفعّلان.
- browser roles لا تملك direct table grants.
- `authenticated` يستطيع تنفيذ RPC الجديد.
- `authenticated` لا يستطيع تنفيذ legacy creation RPC.
- لم تُنشأ Knowledge أو idempotency test records في الإنتاج.

### آخر مشكلتين تم إغلاقهما قبل الدمج

المراجعة النهائية وجدت مشكلتين Medium ثم تم إغلاقهما بالكامل:

1. **Approval committed + response lost**
   - أصبح الخادم يعيد قراءة نفس `source_id` authoritative بعد خطأ الاعتماد.
   - Approved + AI-approved → `made_available`.
   - Confirmed unapproved → `approval_failed_draft_saved`.
   - Unknown state → `approval_status_unknown` دون ادعاء كاذب.

2. **Create committed + response lost + reload/remount**
   - create-attempt UUID أصبح محفوظًا في `sessionStorage`.
   - يبقى نفسه عبر rerender/remount/reload/double-click/transport retry.
   - Save and add another يولد مفتاحًا جديدًا للعملية الجديدة.
   - stale key + different fingerprint يفشل بأمان ويتطلب “Start a new item”.
   - قاعدة البيانات تبقى الضمان النهائي ضد duplicates.

التحقق على الرأس الذي تم دمجه:

- Vitest: 431/431 passed عبر 33 ملفًا.
- Authenticated E2E: 12/12 passed.
- Foundation CI: passed.
- Supabase reset: passed.
- pgTAP: passed.
- Production build: passed.
- Vercel Preview: Ready قبل الدمج.
- Final independent review: لا High ولا Medium findings.

## Meta / WhatsApp

### الحالة

- تم إنشاء Meta Embedded Signup configuration الصحيح المنتهي بـ`1674`.
- التكوين القديم المنتهي بـ`3161` كان incomplete/general وأعاد ordinary Facebook Login.
- التكوين الجديد أعاد authorization code كما هو متوقع.
- Business Verification لدى Meta ما زالت Pending/In progress لأكثر من أسبوع.
- بدأ التأهيل كـ Independent Tech Provider.
- Tech Provider qualification متوقف على Business Verification.
- Advanced Access وApp Review لم يكتملوا بعد.

### Google Cloud / Secret Manager

تم تجهيز Google Cloud سابقًا بالكامل لتخزين بيانات WhatsApp السرية:

- مشروع Google Cloud وإعداداته جاهزة.
- Secret Manager جاهز.
- حساب الخدمة موجود.
- الصلاحيات المطلوبة مُنحت.
- `GOOGLE_SERVICE_ACCOUNT_JSON` موجود على Vercel.
- تم تنفيذ إعادة نشر بعد إعداد Google Cloud سابقًا.

المطلوب عند تدقيق Vercel هو التأكد فقط من متغيرات Meta/WhatsApp الإنتاجية وعدم إعادة بناء Google Cloud من الصفر.

## الإنتاج

- الإنتاج الحالي: `https://dbl-employee-ai.vercel.app`
- `main`: `b14c1fe2e144cb58cd0618501abe3f10c5a88494`
- آخر Knowledge UX المدموج: PR #34
- آخر Knowledge approval/idempotency المدموج: PR #35
- Knowledge Hub الحالي يتضمن:
  - Business information
  - Products/services
  - Policies
  - FAQs
  - Documents
  - Other knowledge
  - AI Employee Readiness بنسبة 30/30/20/20
  - Documents كـ optional enrichment لا تدخل في الجاهزية

## الخطوة التالية حرفيًا

PR #35 مغلق ومكتمل. الخطوة التالية يجب أن تبدأ من `main` الحالي فقط.

الأولوية القادمة قبل توسيع الميزات:

1. تدقيق متغيرات Meta/WhatsApp في Vercel Production/Preview:
   - `META_EMBEDDED_SIGNUP_CONFIG_ID`
   - `META_APP_ID`
   - `META_APP_SECRET`
   - `WHATSAPP_PROVIDER=meta`
   - `WHATSAPP_GRAPH_API_VERSION=v25.0`
   - `WHATSAPP_CREDENTIAL_STORE=gcp_secret_manager`
   - `WHATSAPP_SECRET_MANAGER_PROJECT`
   - `WHATSAPP_VERIFY_TOKEN`
2. عدم إعادة إنشاء Google Cloud؛ البنية موجودة بالفعل.
3. متابعة Meta Business Verification وعدم افتراض اكتمالها حتى يظهر ذلك فعليًا.
4. عند اكتمال Business Verification، استكمال Tech Provider qualification ثم App Review وAdvanced Access.
5. قبل أي feature PR جديد، اقرأ `AI_HANDOFF.md` و`VISION.md` و`ROADMAP.md` لتحديد الأولوية التالية ضمن الرؤية.

## لا تفعل الآن

- لا تعيد فتح أو إعادة تصميم PR #35.
- لا تنشئ migration أو RPC إضافيًا لنفس idempotency دون blocker جديد مثبت.
- لا تغيّر Google Cloud من الصفر؛ Secret Manager والصلاحيات جاهزة.
- لا تفترض أن Business Verification اكتمل حتى يظهر ذلك فعليًا في Meta.
- لا تختبر failure/concurrency production writes إذا كانت الاختبارات المعزولة تكفي.