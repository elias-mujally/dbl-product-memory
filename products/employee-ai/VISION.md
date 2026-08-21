# رؤية DBL Employee AI

آخر تحديث استراتيجي: **2026-08-19**

## الفكرة الجوهرية

DBL Employee AI ليس Chatbot عاديًا، وليس مجرد أداة ردود تلقائية، وليس الهدف النهائي أن يصبح CRM أو ERP جديدًا.

الرؤية هي بناء **موظف رقمي تشغيلي موثوق** يستطيع:

- فهم طلب العميل أو الموظف؛
- استخدام معرفة الشركة المعتمدة؛
- قراءة الأنظمة الحقيقية للنشاط؛
- تجهيز Action تجاري أو تشغيلي؛
- تطبيق سياسات وصلاحيات deterministic؛
- طلب Human Approval عند الحاجة؛
- التنفيذ عبر الأنظمة الخارجية؛
- التحقق من النتيجة؛
- تسجيل ما حدث بصورة قابلة للتدقيق.

الواجهة التسويقية قد تستمر في وصفه ببساطة كـ"موظف AI" أو "Sales Copilot". عبارة **AI Business Execution Layer** هي North Star داخلية أكثر من كونها اسمًا تسويقيًا.

## الوعد الأساسي

> علّم DBL عن نشاطك، اربطه بالأنظمة التي يعمل عليها النشاط، حدّد صلاحياته وحدوده، ثم دعه يجهز وينفذ العمل الحقيقي بأمان وتحت سيطرة الإنسان.

## الفرق بين الاتجاه القديم والجديد

```text
قديم:
Customer asks -> AI answers

اتجاه جديد:
Customer asks
-> AI understands
-> reads real systems
-> prepares action
-> deterministic policy checks
-> human approval when required
-> execute
-> verify
-> reconcile if uncertain
-> audit
```

المحادثة ما تزال مهمة، لكنها **واجهة دخول للعمل** وليست وحدها moat المنتج.

## المبادئ

### 1. النتيجة التجارية أهم من AI vocabulary

DBL يجب أن يبيع نتيجة مفهومة:

- إغلاق فرصة بيع؛
- تقليل عمل يدوي؛
- تجهيز طلب؛
- التحقق من مخزون؛
- تسريع عملية تشغيلية؛
- تقليل أخطاء التنفيذ.

لا يكفي أن نقول "AI Agent" أو "Omnichannel".

### 2. Meta قناة وليست المنتج

WhatsApp/Instagram/Messenger قد تكون قنوات مهمة، لكن DBL يجب ألا ينهار إذا تغيرت منصة واحدة.

### 3. Salla Provider أول، وليست هوية دائمة

Commerce هو أول Vertical مقترح للتحقق، وSalla أول Provider محتمل. لا نبني DBL كملحق وجودي لسلة.

### 4. Knowledge ليست Policy

Knowledge تعطي معنى وسياقًا.

Policy تفرضها البرمجيات deterministically، مثل:

- هل write action مسموح؟
- هل يحتاج موافقة؟
- ما الحد المالي؟
- هل refund ممنوع؟

الـLLM لا يستنتج لنفسه صلاحية من نص Knowledge.

### 5. Human Approval قبل Autonomy

أول Beta حقيقي يجب أن يبدأ بـL1 / Draft-only أو Approval-before-write.

الاستقلالية التلقائية تأتي فقط بعد دليل reliability حقيقي.

### 6. التنفيذ الخارجي يحتاج Verification وReconciliation

إذا كانت نتيجة write action غامضة، لا نعيد المحاولة عميانيًا. نتحقق من النظام الخارجي أولًا.

### 7. Implement first, abstract second

لا universal commerce layer ولا universal workflow engine قبل أن يعمل Provider واحد وAction واحد ويثبتا قيمة.

### 8. العربية تجربة أصلية

- RTL حقيقي.
- واجهة عربية وإنجليزية.
- عزل القيم التقنية LTR.
- دعم لهجات محلية حيث تضيف قيمة حقيقية.

### 9. أمان المستأجرين والأسرار غير قابل للتفاوض

- RLS.
- Workspace isolation.
- Secret Manager.
- no secrets in browser/logs.
- server-authoritative mutations.

## أول بيئة اختبار للرؤية

### Commerce / Sales Execution

الفرضية الحالية:

> DBL يستطيع تحويل intent تجاري إلى Action مفيد وقابل للقياس.

مثال:

```text
customer intent
-> identify product/variant/quantity
-> read inventory/price
-> prepare lowest-risk useful write action
-> merchant approval
-> execute
-> verify
-> record outcome
```

أول write action غير محسوم. قد يكون Draft Order أو prepared checkout أو Action أخرى يثبت Provider API أنها الأنسب.

## ما لم يُثبت بعد

- أن التجار سيدفعون تحديدًا مقابل AI-driven execution.
- أن Sales Execution هو أفضل Wedge.
- أن Salla سيكون Provider الأول فعليًا.
- أن WhatsApp سيكون قناة Beta الأولى إذا بقي Meta عائقًا.
- أن مستوى الاستقلالية يجب أن يتجاوز Human Approval.

هذه فرضيات يجب اختبارها، لا حقائق منتج.

## ما نحتفظ به من المنتج الحالي

Foundation عالية القيمة:

- auth/workspaces/multi-tenancy؛
- Supabase/PostgreSQL/RLS؛
- secure credentials؛
- Knowledge foundation؛
- Contacts foundation؛
- localization؛
- CI/testing discipline؛
- idempotency؛
- Reserve -> Authorize -> Execute -> Observe -> Reconcile pattern؛
- audit concepts.

نعيد استخدام **النمط** حيث يصلح، لا نفترض أن WhatsApp-specific code يصبح execution engine عامًا.

## الرؤية القريبة

إثبات Action واحد مع Merchant واحد:

- pain حقيقي؛
- read access مقبول؛
- write محدود بموافقة بشرية؛
- Provider semantics مفهومة؛
- Action يقلل وقتًا أو يؤثر في إيراد؛
- merchant مستعد للاستمرار أو الدفع.

## رؤية 1–2 سنة إذا نجحت الفرضية

- Commerce Sales/Operations wedge مثبتة.
- عدة Actions موثوقة داخل نفس Vertical.
- Provider ثانٍ يبرر استخراج abstraction حقيقية.
- Human approval وpolicy/audit قويان.
- WhatsApp/web/قنوات أخرى تصبح مداخل إلى نفس execution core بدل منتجات مستقلة.

## رؤية 3–5 سنوات

إذا أثبت Commerce execution قيمته، يمكن أن يتوسع DBL تدريجيًا نحو:

> **AI Operational Layer over the company software stack**

وقد يشمل لاحقًا commerce، CRM، ERP، payments، shipping، internal tools، وقنوات متعددة.

هذه رؤية متعددة السنوات، وليست Scope الـMVP الحالي.

## خارج النطاق حاليًا

- universal execution platform؛
- CRM/ERP كامل؛
- generic chatbot builder؛
- omnichannel expansion بلا دليل؛
- marketplace/plugin ecosystem؛
- multi-agent architecture؛
- universal policy DSL؛
- autonomous financial/legal actions بلا مراجعة؛
- bulk spam؛
- اختلاق knowledge أو provider state؛
- بناء abstraction قبل evidence.

## المقياس الذي يهم الآن

> **هل Merchant حقيقي يريد من DBL Action حقيقيًا، وهل يستطيع DBL تنفيذ هذا Action بأمان؟**

إذا لم يثبت Sales Execution نفسه، نغيّر الـWedge دون اعتبار foundation أو الرؤية بعيدة المدى فاشلة تلقائيًا.
