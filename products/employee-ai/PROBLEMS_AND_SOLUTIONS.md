# DBL Employee AI — Problems and Solutions

هذا الملف هو فهرس مختصر للدروس التشغيلية والهندسية التي ظهرت أثناء بناء المنتج.

## أمثلة على الدروس المحفوظة

- لا تعطّل script security عالميًا لحل build failures.
- webhook verification لا يثبت WABA subscription.
- optional relations لا يجب أن تسقط التطبيق كله، مع الحفاظ على fail-closed للأخطاء الحساسة.
- خلف proxies لا تعتمد على internal request origin.
- Production schema drift يُعالج forward-only reconciliation، لا بتزييف migration history.
- Fake adapters يجب أن تكون مستحيلة في hosted production environments.
- Customer secrets تستحق IAM boundary منفصلًا.
- Embedded Signup يحتاج تصنيف أخطاء دقيق وreadiness موثوق.
- Legal copy يجب أن تصف الواقع لا roadmap مستقبلية.
- Localization تحتاج authority/provenance واضحة.
- Knowledge UX يجب أن تخفي schema التقنية عن المستخدم النهائي.
- idempotency يجب أن تكون server/database enforced لا UI-only.

للتفاصيل الدقيقة، راجع التاريخ السابق للمستودع وGit history وملفات المعمارية المتخصصة.
