# DBL Employee AI — Decisions

هذا الملف هو نقطة الدخول لقرارات المنتج الدائمة.

## قرارات استراتيجية حالية

- DBL Employee AI ليس مجرد chatbot؛ الاتجاه الحالي هو AI Operational Employee / Business Execution Layer.
- Meta قناة وليست المنتج.
- Salla أو أي Provider أول ليس هوية المنتج الدائمة.
- Knowledge لا تمنح صلاحيات؛ Policy الحساسة يجب أن تكون deterministic.
- Human Approval قبل Autonomy.
- External writes تحتاج verification وreconciliation قبل retry عند الغموض.
- Implement first, abstract second.
- لا تبنِ CRM/ERP أو universal workflow engine بلا evidence.

## قرارات تاريخية مهمة

تشمل الذاكرة التاريخية قرارات مثل:

- فصل لغة الواجهة عن لغة رد الموظف؛
- Approved Knowledge فقط تدخل الردود التجارية؛
- Review داخل Workspace العميل؛
- قاعدة البيانات هي ضمان idempotency؛
- Creation وApproval عمليتان منفصلتان؛
- Documents enhancement اختياري؛
- task-oriented Knowledge UX؛
- WhatsApp-first, channel-ready Contacts؛
- UUID داخلي لهوية Contact؛
- account-scope safety؛
- browser writes fail-closed؛
- no merge/delete Contacts in MVP؛
- production preflight gates للترحيلات الحساسة.

عند الحاجة إلى التفاصيل الدقيقة، راجع التاريخ السابق للمستودع أو CHANGELOG وملفات المعمارية المتخصصة.
