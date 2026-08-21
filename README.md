# DBL Product Memory

هذا المستودع هو **الذاكرة المؤسسية العامة لمنتجات Digital Blueprint Lab (DBL)**.

وظيفته حفظ سياق كل منتج أو مبادرة مهمة داخل DBL بحيث يستطيع أي مساعد AI أو عضو فريق جديد فهم:

- ما هو المنتج؟
- ما المشكلة التي يحلها؟
- ما رؤيته الحالية؟
- ما الذي تم تنفيذه فعليًا؟
- لماذا اتخذت القرارات المهمة؟
- ما خارطة الطريق؟
- ما العوائق والمخاطر؟
- أين توقف العمل؟
- ما الذي يجب قراءته قبل استئناف التنفيذ؟

> هذا المستودع ليس مستودع كود، ولا يستبدل Git history أو مستودعات التطبيقات أو بيانات Production كمصدر حقيقة لحالة التنفيذ.

---

## ابدأ من هنا

لأي مساعد AI أو عضو فريق يعمل عبر DBL عمومًا:

1. اقرأ [PORTFOLIO_HANDOFF.md](PORTFOLIO_HANDOFF.md).
2. اختر المنتج المطلوب من مجلد [`products/`](products/).
3. اقرأ `README.md` ثم `AI_HANDOFF.md` داخل مجلد المنتج.
4. لا تستخدم ملفات منتج آخر لتخمين حالة المنتج الحالي.
5. إذا كان السؤال عن التنفيذ الفعلي، تحقق من مستودع الكود الخاص بذلك المنتج.

---

## المنتجات والمبادرات الحالية

### 1. DBL Employee AI

المجلد: [`products/employee-ai/`](products/employee-ai/)

الحالة: منتج قائم وله مستودع كود مستقل وتاريخ تنفيذ فعلي.

الرؤية الحالية: AI Operational Employee / AI Business Execution Layer قيد التحقق.

مستودع التطبيق:
`elias-mujally/dbl-employee-ai`

---

### 2. Legacy Intelligence

المجلد: [`products/legacy-intelligence/`](products/legacy-intelligence/)

الحالة: مبادرة منتج قيد التحقق والتخطيط ولم يبدأ مستودع كود مستقل لها بعد.

الرؤية: Local-First Vendor-Neutral Intelligence & Action Layer فوق الأنظمة القديمة والمحلية، مع تطور مستقبلي نحو Universal Legacy Intelligence Runtime وIndustry Packs.

---

## الهيكل القياسي لكل منتج

كل منتج داخل `products/<product-slug>/` يجب أن يملك على الأقل:

- `README.md` — تعريف المنتج ونقطة الدخول.
- `AI_HANDOFF.md` — ملخص سريع لأي مساعد AI أو عضو فريق جديد.
- `VISION.md` — الرؤية بعيدة المدى والـNorth Star.
- `ROADMAP.md` — ترتيب المراحل والعمل المخطط.
- `DECISIONS.md` — القرارات الدائمة ولماذا اتُخذت.

وعند الحاجة يمكن إضافة:

- `CURRENT_STATUS.md`
- `BLOCKERS.md`
- `CHANGELOG.md`
- `PROBLEMS_AND_SOLUTIONS.md`
- `LATE_DISCOVERED_ISSUES.md`
- `MARKET_STUDY_<DATE>.md`
- `ARCHITECTURE.md`
- ملفات subsystem متخصصة.

ليس كل منتج بحاجة إلى كل ملف منذ اليوم الأول. أنشئ الملف عندما يصبح له محتوى مهم يستحق الحفظ.

---

## قواعد عامة للذاكرة

- لا تخلط حالة منتجين في ملف واحد إلا إذا كان الملف نفسه Portfolio-level.
- حافظ على الفرق بين: hypothesis، market research، decision، implementation state.
- لا تدّعِ أن شيء منفذ لأن وثيقة رؤية قالت إنه مطلوب.
- إذا تعارضت الذاكرة مع الكود أو Production، تحقق من الواقع ثم حدّث الذاكرة.
- لا تنسخ نفس الفقرة عبر عدة ملفات؛ اربط إلى المصدر المرجعي الأفضل.
- لا تضع أسرارًا أو بيانات عملاء حساسة في الذاكرة.

---

## ملفات عامة على مستوى DBL

- [PORTFOLIO_HANDOFF.md](PORTFOLIO_HANDOFF.md) — نقطة دخول عامة لكل المنتجات.
- [PRODUCT_MEMORY_GOVERNANCE.md](PRODUCT_MEMORY_GOVERNANCE.md) — سياسة إدارة الذاكرة.

أي ملفات أخرى في الجذر يجب أن تكون Portfolio-level أو legacy migration artifacts مؤقتة حتى اكتمال النقل إلى مجلدات المنتجات.

---

## إضافة منتج جديد مستقبلًا

عند اعتماد فكرة جديدة كمنتج أو مبادرة تستحق ذاكرة دائمة:

```text
products/
  new-product/
    README.md
    AI_HANDOFF.md
    VISION.md
    ROADMAP.md
    DECISIONS.md
```

ثم أضف المنتج إلى هذا README وإلى `PORTFOLIO_HANDOFF.md`.

الهدف أن يصبح `dbl-product-memory` **ذاكرة Portfolio كاملة لـDBL** وليس ذاكرة مشروع واحد فقط.
