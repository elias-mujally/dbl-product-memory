# DBL Employee AI — Late Discovered Issues

هذا الملف يجمع المشكلات التي ظهرت متأخرًا أثناء الاستخدام الفعلي أو أثناء التدقيق المتقدم ولم تكن واضحة في التصميم الأولي.

أمثلة على الفئات التي يجب حفظها هنا:

- provider/runtime mismatches؛
- credential lifecycle races؛
- production-only schema drift؛
- webhook ownership/cutover confusion؛
- account-scope identity edge cases؛
- discrepancies between test and real WhatsApp environments؛
- deployment/runtime assumptions that failed in production.

الهدف هو منع إعادة اكتشاف نفس الفئة من المشكلات في جولات مستقبلية.
