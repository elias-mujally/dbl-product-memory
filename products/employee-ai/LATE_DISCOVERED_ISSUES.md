# المشاكل ذات الاكتشاف المتأخر

هذا الملف مخصص للمشكلات التي لا تظهر أثناء البناء أو الاختبارات الآلية أو المراجعة المستقلة، ثم تُكتشف لاحقًا أثناء الاستخدام اليدوي الفعلي للمنتج، خصوصًا على Production.

الهدف ليس استبدال `PROBLEMS_AND_SOLUTIONS.md`، بل الفصل بين مرحلتين مختلفتين:

- `LATE_DISCOVERED_ISSUES.md`: مشكلة مكتشفة متأخرًا، ما زالت قيد التشخيص/الإصلاح/التحقق التشغيلي.
- `PROBLEMS_AND_SOLUTIONS.md`: مشكلة فُهم سببها الجذري وحُلّت وتحققت إنتاجيًا، مع حفظ الدرس النهائي.

كل مشكلة هنا يجب أن تنتقل لاحقًا إلى `PROBLEMS_AND_SOLUTIONS.md` بعد تأكيد السبب الجذري ودمج الإصلاح والتحقق الإنتاجي.

---

## 2026-08-12 إلى 2026-08-13 — Production manual testing repair track

تم اكتشاف أربع مشكلات أثناء اختبار يدوي حقيقي للتطبيق على الهاتف. بعد Production Bug Audit وread-only confirmations ودمج PR #39 ثم PR #40، أصبحت بعض الإصلاحات البرمجية منجزة، بينما ما يزال التشغيل الفعلي يحتاج same-number reconnect ثم controlled webhook cutover ثم PR C.

المراجع:

- `events/2026-08-12-meta-webhook-legacy-cloud-run-confirmed.md`
- `events/2026-08-13-pr39-whatsapp-runtime-readiness-merged.md`
- `events/2026-08-13-pr40-embedded-signup-recovery-merged.md`

### LDI-001 — رد WhatsApp الاختباري الثابت ما زال يعمل بدل الرد المتوقع

**الحالة:** Confirmed root cause — awaiting same-number reconnect + controlled webhook cutover.

**المشاهدة:**

عند إرسال رسائل مختلفة إلى رقم WhatsApp المرتبط، يصل الرد نفسه:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

**السبب الجذري المثبت:**

Meta WhatsApp webhook ما زال يشير إلى Cloud Run القديم بدل Vercel الحالي.

التأكيد الإنتاجي أثبت:

- active Meta callback = Cloud Run legacy target;
- Cloud Run service `dbl-employee-ai-git` يرسل 100% من traffic إلى revision قديمة `00024`؛
- serving revision مبنية من source commit يحتوي `receipt-reply.ts` والرد الثابت؛
- recent POST requests إلى `/api/webhooks/whatsapp` وصلت إلى نفس serving revision أثناء نافذة الاختبار اليدوي؛
- current Vercel/main لا يحتوي النص الثابت أصلًا.

**تصحيح مهم بخصوص AI:**

الكود التاريخي للرد الثابت نفسه لا يستدعي AI أو Gemini/Vertex أو grounding. أي استهلاك GCP لاحظه المستخدم أثناء نافذة الاختبار لم يتم تتبع مصدره لأن الأولوية الحالية إصلاح foundation؛ لا يتم تسجيل أن الـ$0.15 كانت AI cost أو نفي ذلك بشكل مطلق بدون billing attribution audit.

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

**الخطر:** High حتى اكتمال الانتقال التشغيلي.

**قرار التشغيل:**

لا يتم تغيير Meta callback الآن. immediate cutover = NO-GO حتى ينجح same-number reconnect ويصبح outbound credential حديثًا وجاهزًا.

---

### LDI-002 — Embedded Signup لا يفتح/يكمل Meta ويعرض فشل تفويض

**الحالة:** PR #40 diagnostics/recovery merged; exact external Meta/browser cause still requires controlled live reproduction.

**المشاهدة الأصلية:**

عند محاولة الربط كان يظهر:

> تعذر إكمال الاتصال
>
> لم تعد Meta نتيجة تفويض مكتملة. ابدأ محاولة الربط من جديد.

**ما كان مثبتًا قبل PR #40:**

- SDK readiness passed;
- prepared state existed;
- `FB.login` was invoked synchronously from the user click;
- callback returned without `authResponse.code`;
- server-side completion did not begin;
- application mapped the outcome to `authorization_incomplete`.

**ما أصلحه PR #40:**

- privacy-safe browser lifecycle diagnostics بدل collapse جميع الحالات إلى generic failure؛
- safe callback/error taxonomy؛
- bounded missing-asset-event recovery؛
- synchronous `FB.login` user gesture محفوظ؛
- duplicate/stale callback protections؛
- one-launch-per-browsing-context invariant؛
- fixed sessionStorage fence `dbl_whatsapp_embedded_signup_context_consumed`؛
- fragment أصبح non-authoritative UX metadata فقط؛
- same-tab fragment/reload/remount/pasted recovery URL لا تعيد launch eligibility؛
- recovery يحتاج genuine `noopener noreferrer` context جديد؛
- sessionStorage failure يفشل مغلقًا قبل Meta launch؛
- Chromium/Firefox/WebKit isolation coverage؛
- testing-stage Meta notice أصبح owner/admin contextual منخفض البروز؛
- same-number reconnect guidance بقي truthful.

Final reviewed head:

`5488ce4485e141323e3f879e1ef28bced46251cf`

Squash SHA:

`a2ba117e78d674b17eea5a8630b2ccc141e5aaf8`

**ما لم يُغلق بعد:**

السبب الخارجي الحقيقي للـno-code callback لا يزال يحتاج controlled Production reproduction. لا يتم تخمين domain/browser/Meta cause بدون evidence.

**الخطوة التالية:**

Controlled Production same-number reconnect بعد تأكيد Vercel Production READY على PR #40 squash SHA وconfig ending `1674`.

---

### LDI-003 — الإرسال اليدوي من صفحة المحادثة يفشل

**الحالة:** Root cause confirmed — PR #39 code fix merged; operational credential transition still pending.

**السبب الجذري المثبت:**

Production connection ما زالت تشير إلى legacy credential location:

`env:WHATSAPP_ACCESS_TOKEN`

بينما Vercel runtime الحالي يعتمد modern Secret Manager/WIF credential path.

**ما أصلحه PR #39:**

- readiness أصبحت server-authoritative ومشتقة من runtime الحالي؛
- legacy connection تبقى Connected تاريخيًا لكنها لا تُعرض كـoutbound-ready؛
- manual/automatic send blocked before Meta عندما credential غير قابلة للاستخدام؛
- manual permanent failure blocks before durable reservation؛
- retries المختلفة لا تنشئ outbound requests/messages/attempts ولا تستهلك quota؛
- late readiness/provider check بقي لحماية TOCTOU؛
- owner/admin يحصلان على guidance لإعادة ربط نفس الرقم؛
- لا token copy/migration ولا account-scope rewrite.

Final reviewed head:

`e7a3d50b7d2b2328cd6bfc7d2c91231c6efc2d7b`

Squash SHA:

`7f5d78ee3441be4cce8c8671992f08cb98b43717`

**ما لم يُغلق بعد:**

Production outbound نفسه لن يصبح operational حتى ينجح same-number Embedded Signup reconnect وينتقل credential reference إلى modern `gcp-sm://...` path، ثم تصبح PR #39 readiness = `ready`.

---

### LDI-004 — Step 5 في إعداد الموظف الذكي يدخل حالة Response Mode غير قابلة للاسترداد

**الحالة:** Confirmed UI/state-machine root cause — awaiting PR C.

**المشاهدة:**

- Automatic يظهر selected ثم disabled؛
- التحويل إلى Review-only لا يسمح لاحقًا باستعادة Automatic؛
- activation يعيد `اختر وضع رد صالحاً`؛
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

**Response test:**

المثبت أنه اكتمل لكنه أعاد `grounded=false`، وليس transport failure. السبب التفصيلي غير محفوظ حاليًا بما يكفي للتمييز بين insufficient knowledge / unsupported answer / grounding rejection أو نتيجة أخرى.

**الخطورة:** High.

**الإصلاح المتوقع:** PR C مستقل بعد إكمال messaging foundation والانتقال التشغيلي الحالي.

---

## اكتشافات إضافية من Production Bug Audit

### Provider diagnostics

PR #39 حسّن provider/readiness taxonomy بدل انهيار حالات متعددة إلى `provider_configuration_missing`.

ما يزال من الممكن زيادة test depth لبعض generic lookup/store failures لاحقًا، لكنها Low وليست correctness blockers.

### AI diagnostics واسعة أكثر من اللازم

Unexpected provider failures قد تنهار إلى `generation_failed`، ما يصعب إثبات failure stage أو cost behavior.

### Simple-reply semantics تحتاج توحيد

Product copy قد يوحي بأن deterministic conversational replies تعمل تلقائيًا، بينما review-only semantics تمنع automatic sending. يجب توحيد copy/runtime contract.

### Readiness authority drift

Onboarding UI وactivation/persisted settings لا تعتمد دائمًا authority واحدة، وهو سبب أوسع من radio bug المحدد.

---

## القرار التشغيلي الحالي

Contacts PR 2 = NO-GO مؤقتًا.

الترتيب الحالي:

1. PR A — WhatsApp credential/runtime readiness — MERGED as PR #39
2. PR B — Embedded Signup Production diagnostics and recovery — MERGED as PR #40
3. **Controlled Production same-number reconnect — NEXT**
4. verify modern Secret Manager credential + PR #39 readiness `ready`
5. Controlled operational Meta webhook cutover من legacy Cloud Run إلى canonical Vercel
6. PR C — Onboarding response-mode state machine
7. Production verification
8. بعدها فقط Resume Contacts PR 2

Immediate webhook cutover remains NO-GO حتى ينجح credential transition ويثبت outbound readiness.

---

## قواعد هذا الملف

عند تحديث أي مشكلة:

1. لا نحول الاشتباه إلى حقيقة دون دليل.
2. نسجل `Observed symptom`, ثم `Confirmed root cause`, ثم `Fix`, ثم `Production verification` كلما تقدمت الحالة.
3. لا نخزن أرقام هواتف حقيقية أو customer IDs أو tokens أو secrets أو provider payloads.
4. بعد إغلاق المشكلة نهائيًا، ننقل الدرس إلى `PROBLEMS_AND_SOLUTIONS.md` ونحتفظ هنا بإشارة إلى الإصلاح/PR المرتبط بها.
