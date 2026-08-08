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
- الرأس النهائي الحالي: `c0421c86bcd2ca87c0ff338a3de77256d061d0e1`
- جميع الفحوص النهائية خضراء.
- درجة الجاهزية الحالية: **99/100**.
- لا توجد High findings.
- لا توجد Medium findings.

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

### آخر إصلاح كان متوقفًا ثم اكتمل

الإصلاح الذي كان محليًا فقط بسبب حد Codex تم دفعه واختباره الآن.

الملفان:

- `components/knowledge-task-form-actions.tsx`
- `tests/e2e/knowledge-idempotency.spec.ts`

السلوك النهائي:

- “Save and add another” يجهز idempotency key جديدًا لعملية إنشاء جديدة فعلًا.
- retry أو النقر المكرر لنفس العملية يحتفظ بالمفتاح نفسه.
- الاختبار يثبت أن عمليتين مقصودتين منفصلتين تنشئان مصدرين مختلفين، دون duplicate داخل العملية نفسها.

التحقق:

- Prettier: passed
- lint: passed
- typecheck: passed
- focused tests: 13/13 passed
- Vitest: 423/423 passed
- git diff --check: passed
- Foundation CI: passed
- Supabase reset: passed
- pgTAP: passed
- Authenticated E2E: passed
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

1. إجراء **مراجعة مستقلة نهائية واحدة** لـPR #35 على الرأس:
   `c0421c86bcd2ca87c0ff338a3de77256d061d0e1`
2. التأكد أن المراجعة لا تجد High أو Medium findings.
3. إذا بقيت الفحوص خضراء، تنفيذ squash merge لـPR #35.
4. انتظار Vercel Production حتى READY.
5. التحقق إنتاجيًا دون بيانات اختبار حقيقية من:
   - approval UX حسب الدور
   - review queue
   - partial-failure wording
   - عدم وجود runtime errors أو 5xx
6. عدم تجربة duplicate production writes إذا كان يمكن الاعتماد على synthetic E2E/database coverage.
7. تحديث ذاكرة المشروع فور الدمج أو عند ظهور blocker جديد.

## لا تفعل الآن

- لا تعيد تصميم idempotency.
- لا تنشئ PR بديل لنفس التغيير.
- لا تغيّر schema أو RPC إضافيًا دون blocker مثبت.
- لا تغيّر Meta أو WhatsApp أو AI behavior أثناء إنهاء PR #35.
- لا تفترض أن Business Verification اكتمل حتى يظهر ذلك فعليًا في Meta.