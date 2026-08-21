# AI Handoff — Legacy Intelligence

## الهدف

هذا الملف يتيح لأي مساعد AI أو مطور فهم مبادرة Legacy Intelligence بسرعة دون قراءة بقية منتجات DBL.

## اقرأ أولًا

1. `README.md`
2. `VISION.md`
3. `ROADMAP.md`
4. `MARKET_STUDY_2026-08-21.md`
5. `MULTI_INDUSTRY_VISION_2026-08-21.md`
6. `DECISIONS.md`

## الوصف الحالي

DBL يدرس بناء **Local-First Legacy ERP Intelligence Layer**: طبقة ذكاء وتشغيل محلية فوق ERP/POS/Accounting/Custom Systems القديمة، تعمل بدون اعتماد دائم على الإنترنت، وتتحول تدريجيًا من Read-only intelligence إلى controlled actions ثم automation.

## المبادئ الثابتة

- لا تبنِ ERP كاملًا أولًا.
- لا تجعل Offline + Arabic + AI هي الـmoat الوحيدة.
- ابدأ بنظام قديم حقيقي واحد وعميل حقيقي واحد.
- Read-only أولًا.
- AI لا يكتب مباشرة في قاعدة البيانات.
- استخدم Typed Actions + Validation + Permission + Preview + Approval + Deterministic Executor + Audit.
- Cloud اختياري، وليس dependency لازمة لاستمرار العمل المحلي.
- Implement first, abstract second.
- Connector knowledge وCanonical Business Model وSchema Intelligence هي أصول دفاعية مستقبلية.
- الرؤية متعددة القطاعات، لكن الـMVP رأسي وضيق.

## الحالة الحالية

لم يبدأ مستودع كود مستقل للمنتج بعد. المرحلة الحالية هي product architecture / memory restructuring / pre-build planning.

## التسلسل المقترح

V1: أول Connector يدوي + Canonical Model أولي + Local read-only intelligence.

V2: فصل mapping عن connector.

V3: Generic SQL Schema Inspector.

V4: AI-assisted semantic mapping.

V5: Capability detection.

V6: Industry Packs وتجربة شبه عامة على أنظمة وقطاعات متعددة.

## تحذير

لا تدّعِ أن أي Capability منفذة قبل التحقق من مستودع الكود المستقبلي الخاص بالمنتج.
