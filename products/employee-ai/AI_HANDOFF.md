# AI Handoff — DBL Employee AI

## الهدف

هذا الملف يتيح لأي مساعد AI أو مطور فهم DBL Employee AI بسرعة دون قراءة منتجات DBL الأخرى.

## اقرأ أولًا

1. `CURRENT_STATUS.md`
2. `STRATEGY_PIVOT_2026-08-19.md`
3. `VISION.md`
4. `ROADMAP.md`
5. `DECISIONS.md`
6. `PROBLEMS_AND_SOLUTIONS.md`
7. `LATE_DISCOVERED_ISSUES.md`
8. `CHANGELOG.md`
9. `BLOCKERS.md`

## الهوية

- المنتج: DBL Employee AI
- المظلة: Digital Blueprint Lab (DBL)
- التطبيق: `elias-mujally/dbl-employee-ai`
- الإنتاج: `https://dbl-employee-ai.vercel.app`
- الذاكرة: `elias-mujally/dbl-product-memory/products/employee-ai`

## الوصف الحالي في سطر واحد

DBL Employee AI يتجه من موظف AI يركز على المحادثة إلى **موظف تشغيلي موثوق** يفهم الطلب، يقرأ أنظمة النشاط، يجهز فعلًا تجاريًا، يطبق سياسات deterministic، يطلب موافقة عند الحاجة، ينفذ، يتحقق، ويسجل النتيجة.

هذه North Star قيد التحقق، وليست تصريحًا لبناء منصة Execution عامة الآن.

## قواعد الثقة

- لا تفترض أن Product Memory تثبت حالة التنفيذ؛ تحقق من مستودع التطبيق.
- لا تفترض أن Production يطابق local branch.
- لا تغيّر Production/Meta/Supabase data دون موافقة صريحة.
- لا تضع secrets في commits أو logs أو الذاكرة.
- Product Memory توثق القرار والسياق؛ التطبيق ومصادر Production هي مصدر الحقيقة للتنفيذ.

## قواعد هندسية ثابتة

- Forward-only migrations.
- لا تعديل migration تاريخية.
- RLS وworkspace isolation لا تُضعف.
- deterministic policy تفرض الأموال والصلاحيات والحالات الحرجة.
- أي external write يحتاج ambiguous/unknown state وreconciliation قبل retry.
- reuse architectural pattern، لا تنسخ implementation خاص بواتساب بلا دليل.
- implement first, abstract second.
- Human approval قبل autonomy.

## ملاحظة

هذا الملف هو handoff مختصر. التفاصيل التاريخية والتنفيذية محفوظة في الملفات المرجعية داخل نفس المجلد.
