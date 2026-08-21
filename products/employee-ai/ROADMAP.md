# DBL Employee AI — Roadmap

آخر تحديث: **2026-08-19**

## المرحلة الحالية

> **Validation + technical spike + evidence -> narrow build decision**

لا نبني Execution Platform عامة قبل إثبات Merchant واحد + Action واحد + Provider واحد.

## المرحلة 1 — Market validation

- مقابلة نحو 10–20 Merchant حقيقيين.
- فهم workflow البيع الفعلي.
- قياس pain، الوقت، willingness-to-pay، والقبول بالـread/write المحدود.

## المرحلة 2 — Provider access + read spike

- تحديد المسار الرسمي للوصول إلى Salla أو Provider أول مناسب.
- اختبار products / variants / inventory / required order data.

## المرحلة 3 — اختيار Action الأول

Action منخفض المخاطر، ذو قيمة اقتصادية، مع Human Approval وverification وidempotency/reconciliation واضحين.

## المرحلة 4 — Wizard-of-Oz / semi-manual prototype

اختبار intent -> proposal -> approve/reject -> execute -> verify مع قياس التصحيح والقيمة.

## المرحلة 5 — Narrow execution beta

- deterministic policies;
- approval-before-write;
- ambiguous outcome handling;
- reconciliation before retry;
- audit trail.

## Parallel tracks

WhatsApp/Meta يبقى مسارًا موازيًا وليس gate لاختبار الفرضية الأساسية.

## North Star بعيدة المدى

```text
one useful action
-> several commerce actions
-> second provider
-> real abstraction
-> commerce operations
-> additional systems
-> AI Operational Layer
```
