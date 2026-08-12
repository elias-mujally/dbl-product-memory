# DBL Product Memory

هذه هي الذاكرة المؤسسية الرسمية لمنتج **DBL Employee AI**.

الهدف من هذا المستودع هو حفظ:

- الرؤية طويلة المدى للمنتج.
- الحالة الحالية الدقيقة.
- تاريخ البناء والقرارات.
- المشكلات التي ظهرت وكيف حُلّت.
- المشكلات التي تُكتشف متأخرًا أثناء الاستخدام الفعلي قبل اكتمال تشخيصها.
- العوائق الخارجية والتقنية.
- نقطة التوقف الحالية لأي مساعد AI أو عضو فريق جديد.

> هذا المستودع ليس مستودع التطبيق البرمجي. مستودع التطبيق الرئيسي هو:
> `elias-mujally/dbl-employee-ai`

## ابدأ من هنا

لأي مساعد AI أو مطور جديد، اقرأ بالترتيب:

1. [AI_HANDOFF.md](AI_HANDOFF.md)
2. [CURRENT_STATUS.md](CURRENT_STATUS.md)
3. [VISION.md](VISION.md)
4. [ROADMAP.md](ROADMAP.md)
5. [DECISIONS.md](DECISIONS.md)
6. [PROBLEMS_AND_SOLUTIONS.md](PROBLEMS_AND_SOLUTIONS.md)
7. [LATE_DISCOVERED_ISSUES.md](LATE_DISCOVERED_ISSUES.md) — مشاكل ظهرت متأخرًا في الاستخدام الفعلي وما زالت قيد التدقيق أو الإصلاح.
8. [CHANGELOG.md](CHANGELOG.md)
9. [BLOCKERS.md](BLOCKERS.md)
10. [PRODUCT_MEMORY_GOVERNANCE.md](PRODUCT_MEMORY_GOVERNANCE.md) — قواعد تطوير الذاكرة نفسها، ADRs، Mermaid، والأتمتة المستقبلية.

وللمهام المعمارية المتخصصة، اقرأ الملف الخاص بالنظام، مثل:

- [CONTACTS_ARCHITECTURE.md](CONTACTS_ARCHITECTURE.md)

## الروابط الأساسية

- التطبيق الإنتاجي: `https://dbl-employee-ai.vercel.app`
- المستودع البرمجي: `https://github.com/elias-mujally/dbl-employee-ai`
- الموقع التسويقي: `https://dblab.site`
- مستودع الذاكرة هذا: `https://github.com/elias-mujally/dbl-product-memory`

## قواعد التحديث

بعد كل PR مهم أو قرار كبير:

- حدّث `CURRENT_STATUS.md`.
- أضف بندًا إلى `CHANGELOG.md`.
- أضف القرار إلى `DECISIONS.md` إذا كان معماريًا أو تجاريًا.
- حدّث `BLOCKERS.md` إذا ظهر أو اختفى عائق.
- أضف المشكلة المفتوحة إلى `LATE_DISCOVERED_ISSUES.md` إذا اكتُشفت متأخرًا في الاستخدام الفعلي ولم يُثبت سببها أو حلها بعد.
- بعد تأكيد السبب والحل والتحقق الإنتاجي، أضف الدرس إلى `PROBLEMS_AND_SOLUTIONS.md` وحدّث حالة المشكلة المتأخرة.
- اتبع [PRODUCT_MEMORY_GOVERNANCE.md](PRODUCT_MEMORY_GOVERNANCE.md) عند إضافة ADRs أو مخططات أو أتمتة للذاكرة.

الذاكرة ليست نسخة ثانية من Git history. نسجل التغييرات والقرارات التي يحتاجها شخص أو مساعد جديد لفهم المنتج واستكماله دون إعادة بناء السياق من الصفر.

## ملاحظة أمنية

ممنوع وضع:

- API keys
- access tokens
- passwords
- Meta app secrets
- Supabase service-role keys
- بيانات عملاء
- أرقام هواتف حقيقية
- معرفات حساسة كاملة

يمكن توثيق وجود السر أو المتغير باسمه فقط، دون قيمته.