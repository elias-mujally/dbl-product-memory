# Product Memory Governance

**Status:** Active portfolio documentation policy

هذا الملف يحدد كيف يجب أن تتطور ذاكرة منتجات DBL مع تعدد المنتجات والمبادرات.

## لماذا يوجد هذا المستودع

`dbl-product-memory` هو الذاكرة المؤسسية الدائمة **لـPortfolio منتجات DBL كامل**، وليس لمنتج واحد فقط.

وظيفته حفظ السياق الذي لا يستطيع الكود وحده شرحه بوضوح:

- ما الذي يحاول كل منتج أن يصبحه؛
- ما الذي تم تنفيذه فعليًا وما الذي لا يزال فرضية؛
- لماذا اتخذت القرارات المهمة؛
- ما البدائل التي نوقشت ورُفضت؛
- ما المشاكل التي ظهرت وكيف حُلّت؛
- ما الذي تم تأجيله عمدًا؛
- أين توقف العمل؛
- وما الذي يحتاجه مساعد AI أو عضو فريق ليستأنف العمل دون إعادة بناء التاريخ من الصفر.

---

## 1. Portfolio-first structure

الهيكل القياسي:

```text
README.md
PORTFOLIO_HANDOFF.md
PRODUCT_MEMORY_GOVERNANCE.md
products/
  product-a/
    README.md
    AI_HANDOFF.md
    VISION.md
    ROADMAP.md
    DECISIONS.md
    ...
  product-b/
    README.md
    AI_HANDOFF.md
    VISION.md
    ROADMAP.md
    DECISIONS.md
    ...
```

الجذر يجب أن يحتوي فقط على ملفات Portfolio-level أو ملفات migration مؤقتة أثناء إعادة الهيكلة.

---

## 2. AI-ready documentation

عند تسليم أي منتج إلى Codex أو Claude أو Gemini أو DeepSeek أو Copilot أو أي مساعد آخر:

1. ابدأ من `README.md` في الجذر إذا كانت المهمة عامة على DBL.
2. اختر المنتج المطلوب من `products/<product>/`.
3. اقرأ `README.md` ثم `AI_HANDOFF.md` داخل مجلد المنتج.
4. اقرأ الملفات المتخصصة للمهمة قبل اقتراح تنفيذ.
5. تحقق من مستودع الكود الخاص بالمنتج قبل اعتبار Product Memory دليلًا على حالة التنفيذ الحالية.

لا يجب على المساعد أن يخلط بين منتجات مختلفة أو يستخدم Roadmap منتج كمرجع لمنتج آخر.

---

## 3. Product folder minimum

كل منتج معتمد أو مبادرة كبيرة تستحق ذاكرة دائمة يجب أن تملك على الأقل:

- `README.md` — تعريف ونقطة دخول.
- `AI_HANDOFF.md` — ملخص عملي سريع.
- `VISION.md` — North Star والرؤية بعيدة المدى.
- `ROADMAP.md` — ترتيب المراحل والأولويات.
- `DECISIONS.md` — القرارات الدائمة.

ملفات اختيارية حسب الحاجة:

- `CURRENT_STATUS.md`
- `BLOCKERS.md`
- `CHANGELOG.md`
- `PROBLEMS_AND_SOLUTIONS.md`
- `LATE_DISCOVERED_ISSUES.md`
- `MARKET_STUDY_<DATE>.md`
- `ARCHITECTURE.md`
- subsystem-specific documents
- `DECISIONS/ADR-*`

---

## 4. Source-of-truth hierarchy

Product Memory تشرح **intent/history/context**.

عندما تكون حالة التنفيذ مهمة:

1. Production الفعلي إن كان موجودًا.
2. مستودع الكود والفرع/commit الحالي.
3. `CURRENT_STATUS.md` داخل مجلد المنتج.
4. ملفات القرار/الرؤية/roadmap لفهم لماذا وصل المنتج إلى هذه الحالة.

إذا تعارضت وثيقة قديمة مع الواقع الحالي، تحقق ثم حدّث الذاكرة.

---

## 5. Separation between product types

يجب الفصل بوضوح بين:

- **Implemented product** — له مستودع كود/Production أو تنفيذ فعلي.
- **Validated concept** — دراسة قوية وفرضية جدية لكن لم يبدأ التنفيذ.
- **Exploratory idea** — فكرة لم تصل بعد إلى مستوى منتج مستقل.
- **Historical/superseded direction** — يحتفظ بها للتاريخ دون تقديمها كحالة حالية.

لا تستخدم لغة توحي بأن concept منفذ أو أن exploratory idea أولوية حالية.

---

## 6. Architecture Decision Records

القرارات المكلفة للعكس أو المتوقع إعادة مناقشتها يجب أن تنتقل تدريجيًا إلى ADRs داخل مجلد المنتج:

```text
products/<product>/DECISIONS/
  ADR-001-....md
```

كل ADR يجب أن يحتوي:

- title؛
- status؛
- date؛
- context/problem؛
- decision؛
- alternatives considered؛
- rejected alternatives؛
- expected consequences/trade-offs؛
- روابط للـPRs أو architecture أو incidents عند الحاجة.

---

## 7. Visual architecture diagrams

استخدم Mermaid عندما تجعل العلاقات أو data flow أو state transitions أو subsystem boundaries أسهل للفهم.

إذا كان الرسم Proposed وليس Approved، ضع بوضوح:

**Conceptual / Pending Technical Audit**

---

## 8. Semi-automated memory

الأتمتة قد تجمع حقائق وتقترح تحديثات، لكنها لا تعيد كتابة product intent بصمت.

مناسب للأتمتة:

- PR metadata؛
- merge SHAs؛
- deployment status؛
- test summaries؛
- closed blockers؛
- changelog suggestions.

لا تعِد كتابة تلقائيًا دون مراجعة:

- `VISION.md`؛
- product strategy؛
- architecture decisions؛
- security assumptions؛
- roadmap priorities؛
- explanations of why decisions were made.

DBL تتبع **semi-automated memory, not autonomous product memory**.

---

## 9. Security rules

ممنوع حفظ:

- API keys؛
- access tokens؛
- passwords؛
- private keys؛
- raw OAuth/OIDC assertions؛
- بيانات عملاء خاصة؛
- customer phone numbers؛
- private message contents؛
- secret provider payloads.

المسموح:

- أسماء المتغيرات السرية؛
- architecture descriptions؛
- masked identifiers عند الحاجة؛
- public PR/commit/deployment identifiers إذا كانت آمنة.

---

## 10. What triggers a memory update

حدّث ذاكرة المنتج عندما يحدث:

- merge مهم؛
- تغير production behavior؛
- blocker جديد أو مغلق؛
- migration تغير data model؛
- قرار product/architecture جديد؛
- قرار قديم تم supersede له؛
- incident يحمل درسًا reusable؛
- تغير next implementation step؛
- market study تغير الفرضية؛
- فكرة أصبحت مهمة بما يكفي لترقيتها إلى product folder مستقل.

لا تسجل كل commit صغير. الذاكرة يجب أن تبقى curated.

---

## 11. Review principle

مراجعات Gemini وDeepSeek وClaude وCodex وCopilot والبشر هي **inputs وليست authority تلقائية**.

عند استخدام مراجعة خارجية:

1. حدد المشكلة الحقيقية وراء الاقتراح.
2. افصل الرأي عن الدليل.
3. قارنه مع المنتج المعني، لا مع DBL بشكل مبهم.
4. اعتمد فقط ما يحسن correctness/safety/clarity/strategy.
5. سجل القرار النهائي كقرار DBL وليس كنسخ لتعليق المستشار.

---

## 12. Adding a new product

عندما تصبح فكرة جديدة مبادرة تستحق مرجعًا مستقلًا:

1. أنشئ `products/<slug>/`.
2. أضف `README`, `AI_HANDOFF`, `VISION`, `ROADMAP`, `DECISIONS`.
3. أضف studies/architecture/history المطلوبة.
4. أضف المنتج إلى root `README.md` و`PORTFOLIO_HANDOFF.md`.
5. اربط مستودع الكود عندما يبدأ التنفيذ.

---

## Current portfolio decision — 2026-08-22

تم اعتماد إعادة هيكلة `dbl-product-memory` من ذاكرة متمحورة حول DBL Employee AI إلى **Portfolio Product Memory** بحيث يملك كل منتج مجلدًا مستقلًا بذاكرته، ويصبح الجذر عامًا لكل منتجات DBL الحالية والمستقبلية.
