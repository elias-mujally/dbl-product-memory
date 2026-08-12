# المشاكل ذات الاكتشاف المتأخر

هذا الملف مخصص للمشكلات التي لا تظهر أثناء البناء أو الاختبارات الآلية أو المراجعة المستقلة، ثم تُكتشف لاحقًا أثناء الاستخدام اليدوي الفعلي للمنتج، خصوصًا على Production.

الهدف ليس استبدال `PROBLEMS_AND_SOLUTIONS.md`، بل الفصل بين مرحلتين مختلفتين:

- `LATE_DISCOVERED_ISSUES.md`: مشكلة مكتشفة متأخرًا، ما زالت قيد التشخيص/الإصلاح.
- `PROBLEMS_AND_SOLUTIONS.md`: مشكلة فُهم سببها الجذري وحُلّت، مع حفظ الدرس النهائي.

كل مشكلة هنا يجب أن تنتقل لاحقًا إلى `PROBLEMS_AND_SOLUTIONS.md` بعد تأكيد السبب الجذري ودمج الإصلاح والتحقق الإنتاجي.

---

## 2026-08-12 — Production manual testing after PR #38 merge

تم اكتشاف أربع مشكلات أثناء اختبار يدوي حقيقي للتطبيق على الهاتف. بعد Production Bug Audit ثم read-only confirmation إضافي، أصبحت بعض الأسباب الجذرية مثبتة وبعضها ما زال مفتوحًا.

مرجع التأكيد الخارجي:

`events/2026-08-12-meta-webhook-legacy-cloud-run-confirmed.md`

### LDI-001 — رد WhatsApp الاختباري الثابت ما زال يعمل بدل الرد المتوقع

**الحالة:** Confirmed root cause — awaiting PR A/PR B ثم controlled webhook cutover.

**المشاهدة:**

عند إرسال رسائل مختلفة إلى رقم WhatsApp المرتبط، يصل الرد نفسه:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

**السبب الجذري المثبت:**

Meta WhatsApp webhook ما زال يشير إلى Cloud Run القديم بدل Vercel الحالي.

التأكيد الإنتاجي أثبت:

- active Meta callback = Cloud Run legacy target.
- Cloud Run service `dbl-employee-ai-git` يرسل **100%** من traffic إلى revision قديمة `00024`.
- serving revision مبنية من source commit يحتوي `receipt-reply.ts` والرد الثابت.
- recent POST requests إلى `/api/webhooks/whatsapp` وصلت إلى نفس serving revision أثناء نافذة الاختبار اليدوي.
- current Vercel/main لا يحتوي النص الثابت أصلًا.

**تصحيح مهم بخصوص AI:**

الرد الثابت نفسه لا يستدعي AI ولا Gemini/Vertex ولا grounding، وبالتالي يستهلك **0 AI tokens**. أي AI events ظهرت لاحقًا كانت عمليات منفصلة وليست جزءًا من الرد الثابت.

**المشكلة المعمارية الأوسع:**

تم إثبات split-runtime:

```text
Inbound Meta webhook
→ legacy Cloud Run
→ historical receipt responder

Current UI/server actions
→ Vercel current main
→ modern outbound / AI / Embedded Signup
```

**الخطر:** High.

**قرار التشغيل:**

لا يتم تغيير Meta callback الآن. immediate cutover = **NO-GO** حتى يصبح outbound/credential path الحديث جاهزًا.

---

### LDI-002 — Embedded Signup لا يفتح Meta ويعرض فشل تفويض

**الحالة:** Open — exact Meta/browser cause not yet confirmed; planned PR B.

**المشاهدة:**

عند محاولة الربط يظهر:

> تعذر إكمال الاتصال
>
> لم تعد Meta نتيجة تفويض مكتملة. ابدأ محاولة الربط من جديد.

**ما تم إثباته:**

- SDK readiness passed.
- prepared state existed.
- `FB.login` was invoked synchronously from the user click.
- callback returned without `authResponse.code`.
- server-side completion did not begin.
- application mapped the outcome to `authorization_incomplete`.
- PR #38 did not introduce the browser sequencing issue.

**ما لم يُثبت بعد:**

السبب الخارجي/المتصفحي الدقيق لعودة callback ناقصًا.

فرضيات ما زالت تحتاج reproduction منضبط:

- Production JSSDK origin/domain authorization state.
- Meta login/session/cookie state.
- browser popup/privacy behavior.

**بانر وضع اختبار موافقات Meta:**

لا يُحذف كليًا ما دامت الموافقات التجارية الخارجية غير مكتملة، لكن يجب تحويله إلى owner/admin contextual notice منخفض البروز بدل warning دائم يوحي بأن التكامل مكسور.

**الأولوية:** Medium / PR B بعد PR A.

---

### LDI-003 — الإرسال اليدوي من صفحة المحادثة يفشل

**الحالة:** Confirmed root cause — awaiting PR A.

**المشاهدة:**

الإرسال اليدوي ينتهي بمحاولة فاشلة ورسالة:

> تعذر إرسال الرسالة. حاول مرة أخرى.

**السبب الجذري المثبت:**

Production connection ما زالت تشير إلى legacy credential location:

`env:WHATSAPP_ACCESS_TOKEN`

بينما Vercel runtime الحالي يعتمد modern Secret Manager/WIF credential path.

المسار يصل إلى provider construction ثم يفشل قبل Meta Graph API request.

Safe failure category:

`provider_configuration_missing`

**الحالة الإنتاجية المؤكدة:**

- connection status = connected.
- receiving account scope موجود ومتسق.
- credential reference موجود لكنه legacy environment-based.
- لا evidence على corruption من PR #38 credential lineage.
- failed manual requests terminal وليست stuck/ambiguous.

**قرار:**

لا blind retry قبل إصلاح credential/runtime readiness أو نجاح same-number Embedded Signup transition إلى Secret Manager credential.

**الخطورة:** High.

---

### LDI-004 — Step 5 في إعداد الموظف الذكي يدخل حالة Response Mode غير قابلة للاسترداد

**الحالة:** Confirmed UI/state-machine root cause — awaiting PR C.

**المشاهدة:**

- Automatic يظهر selected ثم disabled.
- التحويل إلى Review-only لا يسمح لاحقًا باستعادة Automatic.
- activation يعيد `اختر وضع رد صالحاً`.
- response test سبق أن أعاد نتيجة غير صالحة للتفعيل.

**السبب الجذري المثبت:**

Production state يمكن أن يكون:

```text
persisted automatic_replies_enabled = true
current readiness = automatic ineligible
rendered form = Automatic checked + disabled
browser submission = disabled input omitted
server receives mode missing
→ mode_invalid
```

الواجهة تستخدم readiness حية لتحديد eligibility بينما selected radio مشتق من persisted AI settings، ما يسمح بحالة checked-but-disabled مستحيلة عند الإرسال.

الـradio state غير controlled بما يكفي للحفاظ على recovery بعد server errors/reload.

**Response test:**

المثبت أنه اكتمل لكنه أعاد `grounded=false`، وليس transport failure. السبب التفصيلي غير محفوظ حاليًا بما يكفي للتمييز بين insufficient knowledge / unsupported answer / grounding rejection أو نتيجة أخرى.

**الخطورة:** High.

**الإصلاح المتوقع:** PR C مستقل بعد تثبيت WhatsApp foundation أو بالتوازي دون خلط النطاقات.

---

## اكتشافات إضافية من Production Bug Audit

### Provider diagnostics واسعة أكثر من اللازم

عدة حالات مختلفة قد تنهار إلى:

`provider_configuration_missing`

مثل legacy config gap أو credential RPC/WIF/Secret Manager/malformed payload.

يجب إضافة safe stage-specific diagnostics بدون PII/secrets.

### AI diagnostics واسعة أكثر من اللازم

Unexpected provider failures قد تنهار إلى `generation_failed`، ما يصعب إثبات failure stage أو cost behavior.

### Simple-reply semantics تحتاج توحيد

Product copy قد يوحي بأن deterministic conversational replies تعمل تلقائيًا، بينما review-only semantics تمنع automatic sending. يجب توحيد copy/runtime contract.

### Readiness authority drift

Onboarding UI وactivation/persisted settings لا تعتمد دائمًا authority واحدة، وهو سبب أوسع من radio bug المحدد.

---

## القرار التشغيلي الحالي

Contacts PR 2 = **NO-GO مؤقتًا**.

الترتيب المعتمد:

1. **PR A — WhatsApp credential/runtime readiness**
2. **PR B — Embedded Signup Production diagnostics and recovery**
3. **Controlled operational Meta webhook cutover** من legacy Cloud Run إلى canonical Vercel
4. **PR C — Onboarding response-mode state machine**
5. Production verification
6. بعدها فقط Resume Contacts PR 2

Immediate webhook cutover risk: **8/10 High**، لأنه قد يجعل inbound يصل إلى Vercel بينما outbound ما زال غير قادر على الإرسال بسبب legacy credential reference.

---

## قواعد هذا الملف

عند تحديث أي مشكلة:

1. لا نحول الاشتباه إلى حقيقة دون دليل.
2. نسجل `Observed symptom`, ثم `Confirmed root cause`, ثم `Fix`, ثم `Production verification` كلما تقدمت الحالة.
3. لا نخزن أرقام هواتف حقيقية أو customer IDs أو tokens أو secrets أو provider payloads.
4. بعد إغلاق المشكلة نهائيًا، ننقل الدرس إلى `PROBLEMS_AND_SOLUTIONS.md` ونحتفظ هنا بإشارة إلى الإصلاح/PR المرتبط بها.
