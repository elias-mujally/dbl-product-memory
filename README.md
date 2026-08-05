# DBL Product Memory

هذه هي الذاكرة المؤسسية الرسمية لمنتج **DBL Employee AI**.

الهدف من هذا المستودع هو حفظ:

- الرؤية طويلة المدى للمنتج.
- الحالة الحالية الدقيقة.
- تاريخ البناء والقرارات.
- المشكلات التي ظهرت وكيف حُلّت.
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
7. [CHANGELOG.md](CHANGELOG.md)
8. [BLOCKERS.md](BLOCKERS.md)

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
- أضف المشكلة وحلها إلى `PROBLEMS_AND_SOLUTIONS.md` إذا كانت تستحق الاحتفاظ بها.

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