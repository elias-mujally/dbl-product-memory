# العوائق الحالية

آخر تحديث: **2026-08-19**

## 1. Market validation — أكبر عائق معرفي حالي

**الحالة:** غير مثبت بعد.

لدينا دليل أن التجار يدفعون على WhatsApp automation، support، abandoned-cart recovery، order notifications، وAI commerce assistance.

لكن لا يوجد بعد دليل كافٍ أن التجار سيدفعون تحديدًا مقابل:

> AI-driven business execution الذي يحول intent إلى prepared/executed action.

**الأثر:**

يمنع بناء Execution Layer كبيرة بثقة.

**المطلوب:**

- 10–20 merchant interviews؛
- Wizard-of-Oz عند الحاجة؛
- قياس pain، correction rate، time/revenue impact، willingness-to-pay.

---

## 2. Salla developer / partner access

**الحالة:** غير محسومة.

ما ظهر أثناء التسجيل:

- company path يطلب مستندات شركة؛
- personal publishing path يطلب وثيقة عمل حر سعودية؛
- هذا يخلق مشكلة لمؤسس غير سعودي.

**غير مثبت:** أن dev/test API access مستحيل دون هذه الوثائق.

**المطلوب:**

تحديد المسار الرسمي للمطور غير السعودي أو test/development access دون public publication.

**قاعدة:** لا وثائق مزيفة ولا بيانات تجارية مصطنعة.

---

## 3. First meaningful write action غير محسوم

**الحالة:** فرضية.

Candidates تشمل Draft Order وprepared checkout وأفعالًا أخرى منخفضة المخاطر.

**المطلوب:**

اختيار Action من behavior/API حقيقي بعد read spike وmerchant workflow evidence، لا من نقاش معماري فقط.

---

## 4. Meta Business / Tech Provider / approvals

**الحالة:** incomplete / not fully proven.

قد تشمل المتطلبات:

- Business Verification؛
- Tech Provider / Solution Partner qualification؛
- Advanced Access؛
- App Review؛
- coexistence eligibility؛
- required webhook fields.

**الأثر:**

يمنع أو يؤخر live external WhatsApp onboarding.

**ملاحظة:**

هذا لا يجب أن يمنع اختبار core execution hypothesis. استخدم web/internal/semi-manual channel إذا لزم.

---

## 5. WhatsApp real Production connection غير موجود بعد

**الحالة:** current DBL connection is test-only.

ثبت أن Production الحالي يشير إلى Meta Test WABA/Test Number وlegacy credential.

الرقم الحقيقي مختلف ومسجل في WhatsApp Business App.

**الأثر:**

- standard same-number reconnect للاتصال الحالي غير مناسب؛
- real outbound readiness غير متحقق؛
- webhook cutover غير مسموح.

**المطلوب قبل live onboarding:**

- safe test-scope cleanup أو قرار أحدث؛
- confirm coexistence eligibility؛
- required Meta prerequisites؛
- modern Secret Manager credential؛
- controlled verification.

---

## 6. PR #43 — test-scope cleanup ليس جاهزًا تلقائيًا

**الحالة عند آخر تحقق:** Draft / Open / Unmerged.

- head: `b9b4c0ed490aeeba0c4dc65fb5e4eaaf48661396`.
- base أقدم من main الحالي.
- mergeability ليست جاهزة حاليًا.

**الأثر:**

لا يجوز اعتباره طريقًا جاهزًا للحذف.

**المطلوب إذا عاد للمسار:**

- rebase أو rebuild على main الحالي؛
- exact-head database/E2E CI؛
- independent SQL/security review؛
- merge منفصل؛
- cleanup execution authorization منفصل تمامًا.

---

## 7. Legacy Cloud Run webhook ownership

**الحالة:** ما يزال قائمًا تاريخيًا.

Meta inbound webhook يذهب إلى legacy Cloud Run ويشرح الرد الثابت:

> تم استلام رسالتك بنجاح من DBL Employee AI ✅

**التصنيف:** مشكلة تشغيلية منفصلة عن Embedded Signup eligibility.

**Webhook cutover:** NO-GO حتى real connection + modern credential + Vercel inbound/outbound readiness + subscriptions + rollback.

---

## 8. Product focus / scope creep

**الحالة:** خطر دائم.

أكبر خطر حالي هو تحويل North Star إلى build scope ضخم.

لا تبدأ بلا evidence:

- CRM؛
- ERP؛
- universal workflow builder؛
- generic execution framework؛
- additional channels؛
- multi-agent؛
- marketplace؛
- universal policy DSL؛
- provider expansion واسع.

---

## 9. Billing / legal / language/theme

هذه ما تزال غير مكتملة، لكنها **ليست blocker للـvalidation الحالية**.

تعود للأولوية عندما يثبت wedge ويقترب المنتج من paid beta.

تشمل:

- pricing/payment/subscription lifecycle؛
- final legal owner decisions؛
- remaining localization surfaces؛
- language selector؛
- dark mode؛
- analytics/commercial polish.

---

## 10. Docker limitation في بعض بيئات Codex

Supabase reset/pgTAP قد لا يعملان محليًا إذا Docker غير متاح.

GitHub Actions Linux job يبقى gate الرسمي عند الحاجة.

هذا limitation تشغيلي وليس سببًا لخفض متطلبات الاختبار.

---

# عوائق لم تعد يجب أن تقود الأولوية

- Contacts PR 2 ليس blocker استراتيجيًا بحد ذاته.
- PR C onboarding response-mode ليس blocker لاختبار execution thesis.
- تعدد القنوات ليس prerequisite حاليًا.
- إكمال كل SaaS polish ليس prerequisite للمقابلات أو Wizard-of-Oz.

هذه الأعمال قد تعود إذا أصبحت prerequisite مباشرًا لمرحلة مثبتة.
