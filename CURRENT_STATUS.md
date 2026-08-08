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

### PR الحالي

PR المفتوح: **#35 — fix: clarify internal knowledge approval workflow**

- الحالة: Draft، غير مدمج، قابل للدمج.
- الفرع: `codex/knowledge-internal-approval-ux`
- الرأس الحالي بعد آخر إصلاحين: `f09dc02e54a2c3a97805df4400bfc1b81a40826a`
- جميع الفحوص المطلوبة خضراء على هذا الرأس.
- المراجعة الذاتية الحالية: لا High ولا Medium findings.
- **ما زالت مطلوبة مراجعة مستقلة نهائية قصيرة على الإصلاحين الأخيرين قبل الدمج.**

PR #35 يضيف ويثبت:

- توضيح أن المراجعة داخل Workspace العميل وليست بواسطة DBL.
- Owner/Admin: حفظ مسودة أو إتاحة للموظف الذكي.
- Agent: حفظ مسودة أو إرسال للمالك/المدير للمراجعة.
- Viewer: قراءة فقط.
- Review queue للمالك والمدير.
- create idempotency دائم على مستوى قاعدة البيانات.
- partial-failure recovery إذا نجح الحفظ وفشل الاعتماد.
- pending-state UX لمنع النقرات المتكررة.
- role-boundary coverage واختبارات E2E.

### idempotency النهائية

الهجرة المعتمدة داخل PR #35:

- `20260802172230_knowledge_create_idempotency.sql`
- جدول داخلي: `knowledge_source_create_requests`
- unique constraint: `(workspace_id, actor_user_id, idempotency_key)`
- RPC جديد: `create_idempotent_knowledge_source(...)`
- RLS + FORCE RLS.
- `SECURITY DEFINER` مع `search_path=''`.
- request fingerprint باستخدام SHA-256 للمدخلات الدلالية المطبعة.
- الحجز الذري + unique constraint + `SELECT ... FOR UPDATE` للتزامن.
- retry لنفس العملية يعيد نفس source ID بدل إنشاء سجل جديد.

### آخر إصلاحين بعد المراجعة المستقلة

المراجعة المستقلة للرأس السابق `c0421c86bcd2ca87c0ff338a3de77256d061d0e1` وجدت مشكلتين Medium:

1. إذا نجح `approve_knowledge_source` فعليًا لكن ضاعت الاستجابة أو انتهت المهلة، كانت الواجهة قد تعرض أن الاعتماد فشل وأن العنصر ما زال Draft رغم أنه أصبح Approved وAI-usable.
2. مفتاح idempotency كان mount-local؛ فإذا نجح الإنشاء وضاعت الاستجابة ثم حدث remount/reload، يمكن أن يتولد مفتاح جديد ويؤدي retry إلى duplicate.

تم إصلاحهما في الرأس الحالي `f09dc02e54a2c3a97805df4400bfc1b81a40826a`:

- بعد أي خطأ في الاعتماد، يجري الخادم authoritative reconciliation لنفس `source_id`:
  - إذا كان Approved وAI-approved → `made_available`.
  - إذا بقي غير معتمد → `approval_failed_draft_saved`.
  - إذا تعذر تحديد الحالة → `approval_status_unknown` مع رسالة عربية/إنجليزية صريحة لا تدّعي النجاح أو الفشل النهائي.
- مفتاح create-attempt أصبح محفوظًا في `sessionStorage` ومحدد النطاق بواسطة hashes معتمة للـworkspace/user/form flow.
- يبقى المفتاح نفسه عبر rerender/remount/reload/double-click/transport retry.
- يتم مسحه أو تدويره فقط بعد terminal result مؤكد أو عند بدء عنصر جديد مقصود.
- “Save and add another” يولد مفتاحًا جديدًا للعنصر التالي.
- fingerprint conflict لا يُحل بتوليد duplicate؛ بل يتطلب إجراء صريح “Start a new item”.
- لا يتم تخزين Knowledge content أو customer data أو secrets في browser storage، فقط UUID للمحاولة.

التحقق على الرأس الحالي:

- Prettier: passed
- lint: passed
- typecheck: passed
- Vitest: 431/431 passed عبر 33 ملفًا
- production build: passed
- git diff --check: passed
- secret/PII scan: passed
- Foundation CI: passed
- Supabase reset: passed في CI
- pgTAP: passed في CI
- Authenticated E2E: 12/12 passed
- Vercel Preview: Ready

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
- آخر Knowledge UX المدموج: PR #34
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

1. إجراء **مراجعة مستقلة نهائية مركزة فقط** على الإصلاحين الأخيرين في الرأس:
   `f09dc02e54a2c3a97805df4400bfc1b81a40826a`
2. المراجعة يجب أن تتحقق من:
   - approval committed + response lost → reconciliation يعرض الحالة الصحيحة.
   - genuine approval failure → يبقى unapproved مع partial-success recovery صحيح.
   - reconciliation ambiguity → `approval_status_unknown` دون ادعاء كاذب.
   - create commit + response lost + remount/reload + retry → مصدر واحد فقط.
   - sessionStorage key lifecycle وعدم تدويره في retry.
   - Save and add another → مفتاح جديد وعملية جديدة فعلًا.
   - stale key + different fingerprint → رفض آمن وإجراء Start a new item.
3. إذا لم تظهر High أو Medium findings على الرأس نفسه، تحويل PR #35 إلى Ready for Review ثم squash merge.
4. انتظار Vercel Production حتى READY.
5. تنفيذ production smoke verification دون كتابة بيانات اختبار خطرة.
6. تحديث هذه الذاكرة فور الدمج أو ظهور blocker جديد.

## لا تفعل الآن

- لا تدمج PR #35 قبل المراجعة المستقلة الأخيرة على الرأس `f09dc...`.
- لا تعيد تصميم idempotency.
- لا تنشئ PR بديل لنفس التغيير.
- لا تغيّر schema أو RPC إضافيًا دون blocker مثبت.
- لا تغيّر Meta أو WhatsApp أو AI behavior أثناء إنهاء PR #35.
- لا تفترض أن Business Verification اكتمل حتى يظهر ذلك فعليًا في Meta.