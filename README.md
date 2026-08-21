# DBL Product Memory

هذه هي الذاكرة المؤسسية الرسمية لمنتج **DBL Employee AI** ولمبادرات المنتجات الاستراتيجية المهمة في DBL.

الهدف من هذا المستودع هو حفظ الرؤية، الحالة الحالية، القرارات، تاريخ البناء، المشاكل، العوائق، الأفكار الاستراتيجية المهمة، ونقطة الاستئناف لأي مساعد AI أو عضو فريق جديد.

> مستودع التطبيق الرئيسي هو `elias-mujally/dbl-employee-ai`. هذا المستودع يوثق النية والسياق والقرارات، ولا يستبدل Git history أو الكود كمصدر حقيقة للتنفيذ.

## ابدأ من هنا

لأي مساعد AI أو مطور جديد يعمل على **DBL Employee AI**، اقرأ بالترتيب:

1. [AI_HANDOFF.md](AI_HANDOFF.md)
2. [CURRENT_STATUS.md](CURRENT_STATUS.md)
3. [STRATEGY_PIVOT_2026-08-19.md](STRATEGY_PIVOT_2026-08-19.md)
4. [VISION.md](VISION.md)
5. [ROADMAP.md](ROADMAP.md)
6. [DECISIONS.md](DECISIONS.md)
7. [PROBLEMS_AND_SOLUTIONS.md](PROBLEMS_AND_SOLUTIONS.md)
8. [LATE_DISCOVERED_ISSUES.md](LATE_DISCOVERED_ISSUES.md)
9. [CHANGELOG.md](CHANGELOG.md)
10. [BLOCKERS.md](BLOCKERS.md)
11. [PRODUCT_MEMORY_GOVERNANCE.md](PRODUCT_MEMORY_GOVERNANCE.md)

وللمهام المعمارية المتخصصة، اقرأ الملف الخاص بالنظام مثل [CONTACTS_ARCHITECTURE.md](CONTACTS_ARCHITECTURE.md).

## مبادرات منتجات DBL قيد الدراسة

هذه الملفات تسجل أفكار منتجات مهمة بما يكفي لتكون مرجعًا دائمًا، لكنها **لا تعني أن المنتج منفذ أو أنه أصبح أولوية التنفيذ الحالية**.

- [PRODUCT_CONCEPT_LEGACY_ERP_INTELLIGENCE_2026-08-21.md](PRODUCT_CONCEPT_LEGACY_ERP_INTELLIGENCE_2026-08-21.md) — دراسة ورؤية **Local-First Legacy ERP Intelligence Layer**: طبقة ذكاء محلية فوق أنظمة ERP/المحاسبة القديمة، مع خطة MVP، دراسة السوق العربي والعالمي بتاريخ 2026-08-21، المعمارية، المخاطر، الـmoat، الفريق، والتوسع المستقبلي.

## أولوية المصادر عند التعارض

إذا تعارضت ملفات الذاكرة، استخدم هذا الترتيب:

1. `CURRENT_STATUS.md` للحالة التنفيذية الحالية.
2. `STRATEGY_PIVOT_2026-08-19.md` للاتجاه الاستراتيجي الحالي ولماذا تغير.
3. `VISION.md` للرؤية بعيدة المدى.
4. `ROADMAP.md` لترتيب العمل الحالي.
5. الملفات التاريخية و`CHANGELOG.md` لفهم كيف وصلنا إلى هنا.

ملفات `PRODUCT_CONCEPT_*` توثق **مبادرات قيد الدراسة** ولا تتغلب على حالة التنفيذ الحالية أو Roadmap المنتج الرئيسي ما لم يتم اعتمادها لاحقًا بقرار صريح.

إذا تعارضت الذاكرة مع حالة GitHub/Production الفعلية، تحقق من المستودع والإنتاج ثم حدّث الذاكرة. لا تجعل وثيقة قديمة تتغلب على دليل أحدث.

## الحالة الاستراتيجية الحالية

منذ 2026-08-19، لم يعد الافتراض الرئيسي أن ميزة DBL الدفاعية هي مجرد:

> AI Employee + WhatsApp + Knowledge + automation

الاتجاه الجاري التحقق منه هو:

> **AI Operational Employee / AI Business Execution Layer**

أي أن DBL لا يكتفي بالكلام عن العمل، بل يقرأ الأنظمة الحقيقية ويجهز أو ينفذ أعمالًا ضمن سياسات وصلاحيات وموافقة وتحقيق وتدقيق.

هذه **North Star** وليست إذنًا لبناء منصة تنفيذ عامة الآن. أول فرضية اختبار هي Commerce / Sales Execution، مع Salla كأول Provider محتمل إذا توفر وصول رسمي. Merchant evidence يسبق التوسع الهندسي.

## الروابط الأساسية

- التطبيق الإنتاجي: `https://dbl-employee-ai.vercel.app`
- المستودع البرمجي: `https://github.com/elias-mujally/dbl-employee-ai`
- الموقع التسويقي: `https://dblab.site`
- مستودع الذاكرة: `https://github.com/elias-mujally/dbl-product-memory`

## قواعد التحديث

بعد كل PR مهم أو قرار كبير:

- حدّث `CURRENT_STATUS.md` إذا تغيرت نقطة الاستئناف.
- حدّث `ROADMAP.md` إذا تغير ترتيب الأولويات.
- حدّث `VISION.md` فقط عند تغير الرؤية بعيدة المدى.
- أضف بندًا إلى `CHANGELOG.md` للمراحل المهمة.
- أضف القرار إلى `DECISIONS.md` إذا كان معماريًا أو تجاريًا دائمًا.
- حدّث `BLOCKERS.md` إذا ظهر أو اختفى عائق.
- أضف المشكلة إلى `LATE_DISCOVERED_ISSUES.md` إذا ظهرت في الاستخدام الفعلي ولم يغلق سببها وحلها بعد.
- بعد تأكيد السبب والحل والتحقق الإنتاجي، حدّث `PROBLEMS_AND_SOLUTIONS.md`.
- `STRATEGY_PIVOT_2026-08-19.md` سجل قرار استراتيجي، وليس سجل حالة يومي. لا تعد كتابته مع كل PR؛ حدّثه فقط عندما يظهر دليل يغير الفرضية نفسها.
- استخدم `PRODUCT_CONCEPT_<NAME>_<DATE>.md` للمبادرات الكبيرة قيد الدراسة التي سيكون نسيانها مكلفًا، مع تمييز واضح بين الفرضية، الدراسة، والتنفيذ الفعلي.

## ملاحظة أمنية

ممنوع وضع أسرار أو بيانات عملاء أو أرقام هواتف حقيقية أو معرفات حساسة كاملة في Product Memory. يمكن توثيق وجود سر أو متغير باسمه دون قيمته.
