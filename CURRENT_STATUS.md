# الحالة الحالية

آخر تحديث: **2026-08-05**

## ملخص سريع

DBL Employee AI أصبح تطبيق SaaS فعلي متعدد المستأجرين، بواجهة ثنائية اللغة جزئيًا، ونظام معرفة منظم، وذكاء اصطناعي مؤسس على معرفة معتمدة، وتكامل WhatsApp قيد إكمال متطلبات Meta.

الحالة الحالية ليست Prototype بسيطًا. المنتج يملك بنية حقيقية تشمل:

- Authentication وWorkspace onboarding.
- RLS وعزل المستأجرين.
- WhatsApp webhook ingestion.
- Conversations وContacts foundation.
- Knowledge Base ومراجعة واعتماد للمحتوى.
- Grounded AI drafts وautomatic replies.
- AI employee customization.
- Meta Embedded Signup infrastructure.
- صفحات قانونية عامة.
- بنية localization وtheme.
- Knowledge Hub جديد مبني حول تعليم الموظف الذكي.

## أين توقفنا بالضبط؟

### PR الحالي

PR المفتوح: **#35 — fix: clarify internal knowledge approval workflow**

- الحالة: Draft، غير مدمج.
- الفرع: `codex/knowledge-internal-approval-ux`
- آخر رأس مدفوع إلى GitHub: `d79dc723f45499e4b14c93af60e645eff4a018d3`
- كل فحوص الرأس المدفوع خضراء.
- PR يضيف:
  - توضيح أن المراجعة داخل Workspace العميل وليست بواسطة DBL.
  - أدوار واضحة: Owner/Admin ينشران، Agent يرسل للمراجعة، Viewer قراءة فقط.
  - Review queue داخل Knowledge Hub.
  - idempotent Knowledge creation عبر migration وجدول داخلي وRPC جديد.
  - recovery واضح إذا نجح الحفظ وفشل الاعتماد.

### الإصلاح المحلي غير المدفوع

هناك إصلاح أخير تم تنفيذه محليًا لكنه **لم يُدفع إلى GitHub** بسبب بلوغ حد Codex للأوامر ذات الصلاحيات الخارجية:

- `components/knowledge-task-form-actions.tsx`
- `tests/e2e/knowledge-idempotency.spec.ts`

وظيفة الإصلاح:

- عند استخدام “Save and add another”، يجب توليد idempotency key جديد للعملية الجديدة.
- يجب الاحتفاظ بنفس المفتاح فقط عند retry لنفس العملية.
- أضيف اختبار E2E يثبت أن عمليتين مقصودتين منفصلتين تنشئان مصدرين، دون تكرار داخل العملية الواحدة.

التحقق المحلي لهذا الإصلاح:

- Prettier: passed
- lint: passed
- typecheck: passed
- Vitest: 423/423 passed
- git diff --check: passed

لكن لا يجب دمج PR #35 قبل:

1. دفع الملفين.
2. تشغيل CI على الرأس الجديد.
3. مراجعة مستقلة أخيرة.
4. التأكد أنه لا توجد High/Medium findings.
5. squash merge.

## Meta / WhatsApp

### الحالة

- تم إنشاء Meta Embedded Signup configuration صحيح ينتهي بـ`1674`.
- التكوين القديم المنتهي بـ`3161` كان incomplete/general وأعاد ordinary Facebook Login.
- التكوين الجديد يعيد authorization code كما هو متوقع.
- Business Verification لدى Meta ما زالت Pending/In progress.
- بدأ التأهيل كـ Independent Tech Provider.
- Tech Provider qualification متوقف عند Business Verification.
- Advanced Access وApp Review لم يكتملوا.

### أهم استنتاج

المشكلة الأصلية في Embedded Signup لم تكن من DBL فقط. تم إثبات ذلك عبر صفحة minimal مستقلة لا تستخدم Next.js أو React أو Supabase، وأعادت نفس ordinary login behavior مع التكوين القديم. بعد إنشاء config صحيح، عاد authorization code.

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

عند عودة قدرة Codex على الأوامر الخارجية:

1. فتح checkout الحالي لـPR #35.
2. عرض diff للملفين المحليين فقط.
3. إعادة التحقق.
4. commit + push.
5. انتظار GitHub CI وSupabase/pgTAP وE2E وVercel Preview.
6. مراجعة مستقلة نهائية.
7. إذا لم توجد موانع، squash merge.
8. تحقق إنتاجي من:
   - منع duplicates
   - partial approval failure recovery
   - role matrix
   - review queue

## لا تفعل الآن

- لا تدمج PR #35 على الرأس الحالي.
- لا تنشئ PR بديل لنفس التغيير.
- لا تعيد تصميم idempotency.
- لا تغيّر Meta أو WhatsApp أو production data.
- لا تستخدم client-only lock كضمان وحيد ضد التكرار.