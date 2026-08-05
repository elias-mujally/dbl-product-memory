# القرارات الرئيسية

## 1. فصل لغة الواجهة عن لغة رد الموظف

**القرار:**

- `ui_locale` للواجهة.
- `business_profiles.locale` أو إعدادات الموظف للغة ردود العملاء.

**السبب:**

قد يستخدم المدير واجهة إنجليزية بينما يرد الموظف بالعربية، أو العكس.

**النتيجة:**

تم بناء localization provenance وcompatibility fallbacks دون خلطها بمنطق AI reply language.

---

## 2. Knowledge المعتمدة فقط تدخل الردود التجارية

**القرار:**

الـAI لا يستخدم Draft أو Review-required knowledge كحقيقة جاهزة للعميل.

**السبب:**

منع الأسعار والسياسات والمعلومات غير المعتمدة من الوصول للعملاء.

**النتيجة:**

Approval lifecycle يظل حاجزًا أساسيًا، حتى مع تبسيط UX.

---

## 3. Review داخل Workspace العميل، وليس بواسطة DBL

**القرار:**

- Owner/Admin: حفظ أو إتاحة للموظف.
- Agent: حفظ أو إرسال إلى Owner/Admin.
- Viewer: قراءة فقط.

**السبب:**

لا يجب أن يشعر العميل أن DBL يسيطر على علامته التجارية أو يمنحه الإذن لنشر معلوماته.

---

## 4. قاعدة البيانات هي ضمان idempotency

**القرار:**

لا نعتمد على تعطيل الزر أو React state فقط لمنع duplicates.

**السبب:**

Transport retries، concurrent requests، وأكثر من app instance يمكن أن تتجاوز client locks.

**التنفيذ الحالي في PR #35:**

- جدول داخلي `knowledge_source_create_requests`.
- unique `(workspace_id, actor_user_id, idempotency_key)`.
- request fingerprint.
- RPC أمني idempotent.

---

## 5. Creation وApproval عمليتان منفصلتان

**القرار:**

إنشاء Knowledge يبقى منفصلًا عن `approve_knowledge_source`.

**السبب:**

الحفاظ على دورة الاعتماد الحالية، وصلاحياتها، وسلوك الفشل الآمن.

**سلوك الفشل:**

إذا نجح الإنشاء وفشل الاعتماد، يبقى المصدر Draft ويمكن إعادة الاعتماد دون إنشاء مصدر آخر.

---

## 6. Documents تعزيز اختياري وليست شرط readiness

**القرار:**

Readiness الأساسية:

- Business info 30%
- Products/services 30%
- Policies 20%
- FAQs 20%

Documents = optional enrichment.

**السبب:**

مطعم أو متجر صغير قد يكون جاهزًا دون PDF، ولا يجب أن يتوقف عند 85% بلا سبب.

---

## 7. Knowledge UX مبنية حول المهام لا حول schema

**القرار:**

المستخدم يرى:

- عرّف نشاطك.
- أضف منتجاتك.
- أضف سياساتك.
- أضف FAQs.
- ارفع مستندات.

ولا يرى في المسار الطبيعي:

- Knowledge Source
- Source Type
- Priority
- Chunks
- Revision

**السبب:**

صاحب النشاط لا يجب أن يفهم نموذج قاعدة البيانات ليعلّم موظفه.

---

## 8. لا migration لمجرد UX

**القرار:**

نحافظ على schema الحالية عند إمكانية إعادة تنظيم الواجهة فقط.

**استثناء:**

نقبل migration عند وجود ضمان تقني حقيقي لا يمكن تنفيذه بشكل آمن بدونه، مثل create idempotency.

---

## 9. Embedded Signup config الصحيح بديل للتكوين القديم

**القرار:**

استخدام التكوين الجديد المنتهي بـ`1674`.

**السبب:**

التكوين القديم المنتهي بـ`3161` ظهر incomplete وأعاد ordinary Facebook Login بدل WhatsApp Embedded Signup code/event flow.

---

## 10. لا نشر لمحدد اللغة قبل اكتمال المسارات الأساسية

**القرار:**

بناء localization foundation ثم ترجمة shell/auth ثم protected pages، وبعدها selector.

**السبب:**

زر لغة على تطبيق نصف مترجم يعطي تجربة أسوأ من عدم وجود الزر.

---

## 11. لا Dark Mode قبل ترحيل المكونات

**القرار:**

Theme architecture موجودة، لكن selector مؤجل.

**السبب:**

وجود semantic tokens مع مئات الألوان الثابتة قد ينتج contrast سيئًا إذا فُعّل Dark مبكرًا.

---

## 12. Vercel alias هو عنوان التطبيق الحالي

**القرار:**

العنوان الحالي للمنتج:

`https://dbl-employee-ai.vercel.app`

**وليس:**

- `dblab.site`
- `dbl-website-bay.vercel.app`
- Cloud Run tagged URLs

Cloud Run وimmutable Vercel URLs قد تُستخدم للتحقق، لكنها ليست canonical public product origin الحالية.

---

## 13. Gumroad حل تجاري مؤقت خارج التطبيق

الرؤية التجارية السابقة اعتبرت Gumroad حلًا مؤقتًا لاستلام المدفوعات، لا بنية Billing نهائية للـSaaS.

يجب تصميم Subscription/Billing النهائي بشكل مستقل لاحقًا.

---

## 14. أسلوب العمل للـPRs الكبيرة

1. تنفيذ Draft PR.
2. Self-review نقدي.
3. إصلاح High/Medium.
4. Independent final review.
5. Squash merge مع expected head SHA.
6. Production verification.

هذا الأسلوب اكتشف أخطاء لم تكن ستظهر من الاختبارات السطحية وحدها.