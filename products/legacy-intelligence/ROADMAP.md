# Legacy Intelligence — Roadmap

## المرحلة الحالية

Pre-build validation + architecture definition.

## V1 — First real system / demo-ready

- Local Windows runtime.
- Read-only connector لنظام واحد حقيقي.
- Products / Inventory / Customers / Sales.
- Arabic query/search experience.
- Basic reports.
- Canonical model أولي.
- Offline-first behavior.

**تقدير:** 3–5 أسابيع بعد بدء التنفيذ الفعلي.

## V2 — Mapping separation

- فصل schema mapping عن connector implementation.
- جعل mapping artifact قابلًا للاختبار وإعادة الاستخدام.
- تحسين compatibility/version handling.

**تقدير تراكمي:** 5–7 أسابيع.

## V3 — Generic SQL Schema Inspector

- اكتشاف tables / columns / relationships / data types.
- metadata collection آمن.
- connector diagnostics.

**تقدير تراكمي:** 8–11 أسبوعًا.

## V4 — AI-assisted Semantic Mapping

- اقتراح معنى الجداول والحقول.
- confidence scores.
- human confirmation.
- عدم الاعتماد على AI وحده في business semantics الحساسة.

**تقدير تراكمي:** 12–16 أسبوعًا.

## V5 — Capability Detection

- بناء Business Capability Map.
- اكتشاف قدرات مثل Sales / Inventory / Purchasing / Receivables / Patients / Appointments حسب schema الموثق.
- تفعيل الأدوات المناسبة فقط.

**تقدير تراكمي:** 16–21 أسبوعًا.

## V6 — Industry Packs

- Industry-specific actions, analytics, views, rules.
- أول packs مرشحة: Distribution/Retail، ثم Pharma، ثم Clinic Operations أو قطاعات أخرى حسب الدليل السوقي.
- اختبار onboarding شبه عام لأنظمة متعددة.

**تقدير تراكمي:** 22–30 أسبوعًا.

## قاعدة التنفيذ

لا تنتظر V6 للبيع. الهدف أن تكون V1 قابلة للعرض، ثم pilot حقيقي، ثم paid pilot بمجرد ثبات القيمة والأمان.
