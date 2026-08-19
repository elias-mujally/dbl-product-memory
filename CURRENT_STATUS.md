# الحالة الحالية

آخر تحديث: **2026-08-19**

## الملخص التنفيذي

DBL Employee AI في مرحلة **إعادة توجيه استراتيجية مضبوطة**.

المنتج المنفذ تاريخيًا هو SaaS لموظف AI يعتمد على Knowledge معتمدة، WhatsApp، review/automatic reply modes، Contacts foundation، multi-tenancy، secure credentials، وطبقة اختبار/أمان قوية.

الاتجاه الجاري التحقق منه الآن أوسع وأعمق:

> **AI Operational Employee / AI Business Execution Layer**

أي أن DBL لا يكتفي بالرد، بل يقرأ الأنظمة الحقيقية ويجهز أو ينفذ أعمالًا ضمن policy deterministic، موافقة بشرية، verification، reconciliation، وaudit.

Commerce / Sales Execution هو أول Wedge مقترح فقط، وSalla أول Provider محتمل إذا توفر وصول رسمي. لا يوجد قرار ببناء منصة Execution عامة الآن.

المرجع الاستراتيجي: `STRATEGY_PIVOT_2026-08-19.md`.

---

## Current mode

> **Planning freeze -> validation + technical spike -> evidence -> narrow build decision**

الأولوية لم تعد إضافة features إلى chatbot/omnichannel stack. الأولوية هي إثبات أن Merchant حقيقي يريد Business Action حقيقي ويمكن لـDBL تنفيذه بأمان.

### المطلوب قبل build كبير

1. حل مسار الوصول الرسمي إلى Salla أو Provider أول مناسب.
2. مقابلة نحو 10–20 Merchant حقيقيين في فئات تعتمد على pre-purchase conversation.
3. إجراء provider read spike على products/variants/inventory عندما يتوفر الوصول.
4. اختيار أقل write action مخاطرة بناءً على API حقيقي، لا افتراض نظري.
5. اختبار Wizard-of-Oz أو prototype ضيق مع Human Approval.
6. البناء فقط إذا ظهرت قيمة اقتصادية ورغبة فعلية في الاستخدام/الدفع.

---

## الحالة البرمجية الحالية

### Application main

آخر commit معروف على `main` وقت مزامنة الذاكرة:

`b9d891a30040151ff2105ed219d67720af0f0b7c`

وهو commit توثيقي لإزالة نسخة strategy record وُضعت بالخطأ في مستودع التطبيق. آخر functional WhatsApp merge قبله هو:

`ede862e60bf1ace67a1e3fbb49d1e3f3d4bc7caf` — PR #42.

تحقق من GitHub عند أي مهمة جديدة؛ لا تعتبر SHA هنا ثابتًا.

### WhatsApp PRs المهمة المدمجة

- PR #38: Account-Scope Lifecycle Guard.
- PR #39: Credential / Runtime Readiness.
- PR #40: Embedded Signup Production Diagnostics & Recovery.
- PR #41: privacy-safe asset-event diagnostics.
- PR #42: split malformed asset-event shape diagnostics.

هذه السلسلة أغلقت عدة races وحمت account scope والcredential lifecycle، لكنها لم تجعل الرقم الحقيقي Production-ready.

### PR #43 — Test WhatsApp Scope Cleanup

آخر تحقق:

- Draft / Open / Unmerged.
- head: `b9b4c0ed490aeeba0c4dc65fb5e4eaaf48661396`.
- base قديم مقارنة بـmain الحالي.
- الغرض: migration/operator-only cleanup mechanism خاملة لا تنفذ الحذف تلقائيًا.

**لا تدمج PR #43 كما هو دون إعادة base/CI/review.**

والأهم: حتى دمج PR #43 لا يعني إذنًا بتنفيذ cleanup على Production. أي cleanup يحتاج preflight + snapshot + restore rehearsal + explicit authorization منفصل.

---

## WhatsApp Production — ما ثبت فعليًا

### الاتصال الحالي

ثبت أن اتصال DBL الحالي في Production يشير إلى:

- Meta Test WhatsApp Business Account.
- Meta Test Number.
- legacy environment credential.
- test-only customer/contact/conversation/message history.

البيانات المرتبطة به التي تم تدقيقها كانت كلها اختبارية من المالك، ولا يوجد دليل على real customer activity داخل هذا scope.

### الرقم الحقيقي

الرقم الحقيقي مختلف عن test number ومسجل كـWhatsApp Business App number.

Meta تعرض أنه مسجل كحساب WhatsApp بالفعل، ولذلك Standard Cloud API registration / standard Embedded Signup ليس المسار الصحيح له في حالته الحالية.

### الاتجاه المفضل

إذا بقيت رغبة المالك في استخدام WhatsApp Business App مع DBL على نفس الرقم، فالاتجاه المفضل هو **WhatsApp Business App Coexistence**، بشرط إثبات أهلية Meta والحصول على المتطلبات اللازمة.

Coexistence لم يُنفذ بعد في DBL.

### Multi-connection

بعد تدقيق test scope، تبين أن full Multi-Connection Foundation يمكن تأجيله إذا تم تنظيف test scope بأمان ثم أصبح الرقم الحقيقي هو الاتصال الوحيد في MVP.

هذا قرار تكتيكي لتقليل التعقيد، وليس قرارًا بإلغاء multi-connection مستقبلًا.

---

## Split runtime / legacy Cloud Run

Meta inbound webhook ما يزال تاريخيًا موجهًا إلى legacy Cloud Run الذي يحتوي الرد الثابت القديم:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

هذا يفسر الرد الثابت وهو **مشكلة تشغيلية منفصلة عن Embedded Signup eligibility**.

Webhook cutover يظل **NO-GO** حتى:

- وجود اتصال Production حقيقي؛
- modern credential في Secret Manager؛
- outbound readiness مثبت؛
- Vercel inbound verification مكتمل؛
- event subscriptions المطلوبة جاهزة؛
- rollback plan معتمد.

---

## Contacts

Contacts Foundation PR 1 مدموجة وموجودة.

لكن Contacts PR 2 لم يعد هو next product priority تلقائيًا. لا يبدأ إلا إذا أصبح prerequisite مباشرًا لفرضية Execution/Commerce أو عاد المنتج إلى مسار يحتاجه.

الـContacts الحالية لا تُهدم؛ تعتبر جزءًا من foundation القابلة لإعادة الاستخدام.

---

## ما تم تجميده استراتيجيًا

حتى إثبات Execution hypothesis، لا توسع:

- WhatsApp broadcast/campaigns؛
- chatbot builders؛
- قنوات إضافية؛
- CRM/ERP داخل DBL؛
- multi-agent orchestration؛
- universal workflow builder؛
- marketplace/plugins؛
- universal policy DSL؛
- multi-model routing غير الضروري؛
- Zid/Shopify/WooCommerce قبل provider evidence.

المبدأ:

> **Preserve infrastructure, replace product emphasis.**

---

## العوائق الخارجية الحالية

### Meta

- Business Verification / Tech Provider / Advanced Access / coexistence eligibility ما تزال غير مكتملة أو غير مثبتة بما يكفي للإطلاق الخارجي.
- Meta لا يجب أن تمنع اختبار core execution thesis؛ استخدم قناة web/internal/semi-manual إذا لزم.

### Salla

- مسار الشركة يطلب وثائق شركة.
- مسار الشخص للنشر يطلب وثيقة عمل حر سعودية بحسب ما ظهر في التسجيل.
- لم يثبت بعد أن non-Saudi developer لا يستطيع الحصول على dev/test access.

المطلوب: تحديد المسار الرسمي للمطور غير السعودي أو الوصول التجريبي دون نشر عام. لا تستخدم مستندات مزيفة.

### Market evidence

الدليل الحالي يثبت دفع التجار لأتمتة مجاورة، لكنه لا يثبت بعد willingness-to-pay لـAI-driven business execution.

هذا هو أكبر blocker معرفي للمنتج الآن.

---

## القرار التنفيذي الحالي

لا تبنِ Operational Layer كاملة.

الهدف التالي هو:

> **إثبات أن Merchant واحدًا يريد Action واحدًا حقيقيًا من DBL وأن DBL يستطيع تجهيز/تنفيذ هذا Action بأمان.**

Merchant evidence الآن أهم من المزيد من advisor reports.

---

## لا تفعل الآن

- لا تعمل blind reconnect للـTest WABA.
- لا تستبدل Test WABA/phone identifiers بالرقم الحقيقي داخل نفس historical scope.
- لا تنفذ PR #43 cleanup على Production دون authorization مستقل.
- لا تعمل webhook cutover.
- لا تبدأ general multi-connection فقط لأن architecture المستقبلية قد تحتاجها.
- لا تبدأ Contacts PR 2 أو PR C كأولوية تلقائية.
- لا تبنِ Salla abstraction عامة قبل provider spike حقيقي.
- لا تجعل WhatsApp blocker سببًا لتأجيل merchant validation.
