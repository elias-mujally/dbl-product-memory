# Legacy Intelligence — Current Status

آخر تحديث: **2026-08-22**

## الحالة

**BUILD STARTED — V1 kickoff**

تم الانتقال رسميًا من مرحلة الدراسة والتخطيط إلى مرحلة التنفيذ.

## الهدف التنفيذي الحالي

بناء أول نسخة Demo-ready من المنتج خلال V1، مع Scope ضيق:

- تطبيق Windows محلي.
- Offline-first.
- Read-only.
- Connector واحد فقط في البداية.
- Canonical Business Model أولي.
- دعم Products / Inventory / Customers / Sales.
- واجهة عربية بسيطة.
- Query/Search بلغة طبيعية فوق البيانات الفعلية.
- Basic reports.
- لا Voice.
- لا WhatsApp.
- لا Write Actions في V1.
- لا Multi-industry implementation في V1.

## أول قرار هندسي قبل الكود

نحتاج مستودع كود مستقل للمنتج. الاسم المقترح:

`elias-mujally/dbl-legacy-intelligence`

Product Memory يبقى منفصلًا في:

`elias-mujally/dbl-product-memory/products/legacy-intelligence/`

## أول مسار بناء

1. إنشاء repository مستقل.
2. Scaffold لتطبيق Windows local-first.
3. تعريف boundaries واضحة بين Desktop UI / Core / Connectors / Canonical Model / AI.
4. بناء Demo ERP dataset أولي إذا لم يتوفر ERP حقيقي في أول يوم.
5. بناء أول read-only SQL connector.
6. ربط Products / Inventory / Customers / Sales بالـCanonical Model.
7. إضافة deterministic query layer.
8. إضافة AI intent parsing كطبقة اختيارية فوق query layer، وليس كمصدر حقيقة.
9. بناء واجهة عربية demo-ready.
10. اختبار Offline behavior والنتائج على بيانات حقيقية أو شبه حقيقية.

## قاعدة السلامة

الـLLM لا ينفذ SQL حر على قاعدة العميل ولا يكتب في ERP في V1.

المسار المستهدف:

`User Query -> Intent/Query Plan -> Validated Read Operation -> Connector -> Canonical Result -> Answer`

## حالة المستودع

حتى 2026-08-22 لا يوجد repository مستقل للمنتج ضمن GitHub connector المتاح. موصل GitHub الحالي يستطيع العمل على مستودعات قائمة لكنه لا يوفّر إنشاء Repository جديد من الصفر.

بمجرد إنشاء المستودع الفارغ، يبدأ implementation commit الأول مباشرة.

## Milestone التالي

**Repository initialized + V1 architecture scaffold committed.**
