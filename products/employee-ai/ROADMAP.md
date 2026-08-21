# خارطة الطريق

آخر تحديث: **2026-08-19**

## المبدأ الحاكم

المرحلة الحالية ليست Feature Expansion.

المرحلة الحالية:

> **Validation + technical spike + evidence -> narrow build decision**

لا نبني Execution Platform عامة قبل إثبات Merchant واحد + Action واحد + Provider واحد.

---

# المرحلة 0 — Memory / strategy synchronization

الحالة: **مكتملة في 2026-08-19**

- اعتماد `STRATEGY_PIVOT_2026-08-19.md` كمرجع استراتيجي فعال.
- مزامنة `README`, `AI_HANDOFF`, `CURRENT_STATUS`, `VISION`, `ROADMAP`, `BLOCKERS` مع الاتجاه الجديد.
- منع الملفات القديمة من إعادة المشروع تلقائيًا إلى chatbot/omnichannel roadmap.

---

# المرحلة 1 — Market validation

الحالة: **الأولوية الأولى**

## الهدف

إثبات أو قتل فرضية Commerce / Sales Execution قبل build كبير.

## المطلوب

مقابلة نحو 10–20 Merchant حقيقيين، ويفضل في:

- fashion / abayas؛
- shoes؛
- beauty / perfumes؛
- accessories؛
- selected electronics.

## الأسئلة الأساسية

- لماذا يتواصل العميل قبل الشراء؟
- من يرد اليوم؟
- ماذا يحدث بعد اقتناع العميل؟
- هل ينشئ الموظف طلبًا أو يرسل link يدويًا؟
- ما أكثر خطوة تستهلك وقتًا؟
- أين تضيع المبيعات؟
- ما الأدوات المدفوعة حاليًا؟
- هل merchant يقبل read access؟
- هل يقبل write action محدودًا مع Human Approval؟
- أيهما أفضل في workflow الحقيقي: Draft Order أم prepared checkout أم شيء آخر؟
- ما الذي سيجعل merchant يحذف المنتج بسرعة؟
- ما قيمة الحل إذا أثبت توفير وقت أو إيراد؟

## مخرج المرحلة

واحد من:

- Sales Execution pain مثبت؛
- Wedge مختلفة داخل commerce؛
- commerce ضعيف وننتقل لفرضية أخرى.

لا يعتبر عدم نجاح Sales Execution فشلًا للfoundation.

---

# المرحلة 2 — Provider access + read spike

الحالة: **بالتوازي مع validation**

## Salla

المطلوب أولًا:

- تحديد المسار الرسمي لمطور/مؤسس غير سعودي؛
- معرفة هل dev/test API access متاح دون public app publication؛
- عدم استخدام وثائق شركة أو عمل حر غير صحيحة.

إذا توفر الوصول:

### Read spike

اختبار موثوق لـ:

- products؛
- variants؛
- inventory؛
- price؛
- order data فقط عند الحاجة.

## قاعدة

لا Provider abstraction عامة في هذه المرحلة.

نفذ Salla مباشرة، ثم استخرج abstraction عندما يظهر Provider ثانٍ.

## مخرج المرحلة

- provider access مثبت أو مستبعد؛
- read semantics موثوقة؛
- limitations موثقة؛
- candidate write actions مستخرجة من API الحقيقي.

---

# المرحلة 3 — اختيار Action الأول

لا يُحسم قبل المرحلتين 1 و2.

Candidates تشمل:

- Draft Order؛
- prepared/prefilled checkout؛
- cart recovery action؛
- another low-risk provider-supported action.

## معايير الاختيار

Action الأول يجب أن يكون:

- meaningful اقتصاديًا؛
- low-risk نسبيًا؛
- قابلًا للموافقة البشرية؛
- قابلًا للتحقق بعد التنفيذ؛
- idempotency/reconciliation ممكنة؛
- لا يحتاج universal workflow engine.

---

# المرحلة 4 — Wizard-of-Oz / semi-manual prototype

قبل full automation:

```text
merchant/customer intent
-> DBL/assistant interprets
-> provider data read
-> action proposal
-> merchant approve/reject
-> manual or narrow controlled execute
-> verify outcome
-> measure correction/time/value
```

## القياسات

- كم مرة احتاج merchant تصحيح proposal؟
- كم وقت وفر؟
- هل workflow أسرع من manual؟
- هل merchant يثق في Human Approval model؟
- هل يعود لاستخدامه؟
- هل يدفع؟

---

# المرحلة 5 — Narrow execution beta

تبدأ فقط إذا evidence إيجابي.

## Architecture minimal

ابدأ بنموذج صغير مثل:

- `execution_request`
- `execution_event` فقط إذا أثبتت الحاجة.

## قواعد

- deterministic policies؛
- L1 / approval-before-write؛
- explicit ambiguous outcome؛
- reconcile before retry؛
- audit trail؛
- no automatic high-risk execution.

---

# WhatsApp / Meta track — parallel, not gating PMF

WhatsApp يظل قناة مهمة لكنه لا يجب أن يوقف validation.

## الوضع الحالي

- current Production connection = Test WABA/Test Number.
- real number = WhatsApp Business App number.
- standard Embedded Signup ليس المسار الصحيح للرقم الحقيقي.
- coexistence هو direction محتمل إذا Meta eligibility متوفرة.
- legacy Cloud Run webhook ما يزال قائمًا.

## PR #43

Draft cleanup mechanism للـtest scope.

لا تدمج أو تنفذ تلقائيًا. يحتاج rebase/exact-head CI/security review جديد قبل أي قرار.

## قبل live real-number onboarding

- تنظيف test scope بأمان أو قرار أحدث معتمد؛
- confirm coexistence eligibility؛
- satisfy required Meta permissions/partner gates؛
- modern Secret Manager credential؛
- controlled verification.

## webhook cutover

يبقى NO-GO حتى real connection + inbound/outbound readiness + rollback plan.

إذا تأخر Meta، استخدم web/internal/semi-manual channel لاختبار execution thesis.

---

# Foundation التي نحافظ عليها

لا نهدم بلا سبب:

- authentication؛
- workspaces/multi-tenancy؛
- Supabase/PostgreSQL؛
- RLS/security boundaries؛
- credential handling؛
- Knowledge foundation؛
- Contacts foundation؛
- localization؛
- CI/tests؛
- idempotency/reconciliation؛
- audit patterns.

---

# مؤجل حتى دليل جديد

- Contacts PR 2 كأولوية مستقلة؛
- PR C onboarding response-mode؛
- Analytics expansion؛
- subscription polish؛
- Instagram/Facebook/TikTok channels؛
- broadcast campaigns؛
- generic chatbot builder؛
- CRM/ERP؛
- multi-agent؛
- workflow builder؛
- marketplace/plugins؛
- universal policy DSL؛
- multi-model routing؛
- provider expansion قبل evidence.

قد يعود أي منها إلى الأولوية إذا أصبح prerequisite مباشرًا للـwedge المثبتة.

---

# بوابات القرار

## GO للبناء

إشارات إيجابية:

- pain متكرر في مقابلات التجار؛
- workflow يدوي واضح؛
- merchant يسمح read access؛
- merchant يسمح limited write مع approval؛
- provider API يدعم Action مفيدًا؛
- correction rate منخفض بما يكفي؛
- وقت/إيراد محسّن؛
- willingness to continue/pay.

## Pivot للـWedge

إذا:

- merchant لا يريد action حتى مع approval؛
- workflow يصبح أبطأ؛
- conversation ليست مدخلًا طبيعيًا؛
- API لا يسمح execution مفيدًا؛
- القيمة الاقتصادية صغيرة؛
- pain مختلف يتكرر بقوة.

---

# North Star بعيدة المدى

إذا أثبت Commerce execution نفسه:

```text
one useful action
-> several commerce actions
-> second provider
-> real abstraction
-> commerce operations
-> additional systems
-> AI Operational Layer
```

هذه ليست خطة تنفيذ فورية.

## تعريف milestone الحالي

> **Merchant واحد يريد Action واحدًا، وDBL يستطيع تنفيذ هذا Action بأمان.**
