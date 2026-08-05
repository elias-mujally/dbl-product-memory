# العوائق الحالية

آخر تحديث: 2026-08-05

## 1. Codex external-command limit

**الحالة:** Blocking مؤقتًا لـPR #35.

**الوضع:**

- تم تنفيذ آخر إصلاح محليًا.
- تعذر commit/push لأن بوابة الأوامر الخارجية وصلت حد الاستخدام.
- يتجدد الحد بعد نحو أسبوع من تاريخ التوقف.

**لا يوجد حل آمن بديل مطلوب الآن.**

**عند التجدد:**

- commit + push للملفين المحليين.
- CI.
- final review.
- merge إذا كانت النتائج خضراء.

---

## 2. Meta Business Verification

**الحالة:** Pending / In progress.

**الأثر:**

يمنع أو يؤخر:

- إكمال Independent Tech Provider qualification.
- App Review.
- Advanced Access.
- Customer production onboarding.

**المطلوب:**

انتظار Meta، وعدم الضغط على resend إلا عند وجود دليل أن الطلب عالق أو طلب Meta ذلك.

---

## 3. Tech Provider qualification

**الحالة:** بدأ، 0/2 عند آخر تحقق، ومتوقف على Business Verification.

**لاحقًا:**

- مراجعة app settings.
- screencasts.
- App Review materials.
- permission justifications.

---

## 4. WhatsApp Advanced Access

**غير مكتمل:**

- `whatsapp_business_management`
- `whatsapp_business_messaging`

الحالة السابقة: Ready for testing فقط.

**الأثر:**

Admin/developer testing قد يعمل ضمن حدود، لكن external customer production onboarding غير جاهز.

---

## 5. Embedded Signup FINISH/asset event

**الحالة:** غير محسومة نهائيًا بعد config الجديد.

تم إثبات:

- authorization code يعود مع config 1674.
- config القديم كان خاطئًا.

لم يُثبت بعد end-to-end customer asset grant مع WABA/phone metadata في الإنتاج.

**بعد Meta approval:**

إعادة اختبار المسار الكامل حتى Complete/FINISH، ثم backend exchange والتفعيل.

---

## 6. Legal owner decisions

لم تُحسم نهائيًا:

- legal entity name.
- governing jurisdiction.
- dispute terms.
- exact retention periods.
- long-term privacy/deletion contact.
- finalized versioned Terms acceptance.

الصفحات الحالية محافظة وصادقة، لكنها ليست بديلًا عن legal review عند الإطلاق التجاري الواسع.

---

## 7. Billing/payment

لا يوجد Billing system نهائي داخل المنتج.

المطلوب لاحقًا:

- pricing.
- payment provider.
- plan limits.
- subscriptions.
- invoices.
- failed payment lifecycle.

Gumroad حل مؤقت خارج التطبيق، وليس architecture نهائية.

---

## 8. Language selector وDark Mode

البنية موجودة، لكن controls مؤجلة لأن:

- بعض الصفحات المؤجلة لم تترجم بالكامل.
- Dark mode يحتاج broad component color migration.

إظهار controls الآن قد يكشف تجربة ناقصة.

---

## 9. Docker غير متوفر محليًا في بيئة Codex السابقة

الأثر المتكرر:

- Supabase reset/pgTAP لا يعملان محليًا.

المعالجة:

- GitHub Actions Linux job هو gate الرسمي.
- authenticated E2E تستخدم local disposable Supabase داخل CI.

هذا ليس blocker للدمج إذا CI الأخضر يغطيه، لكنه limitation يجب ذكره دائمًا.

---

## 10. Product areas غير مكتملة

- Contacts route/UI.
- Analytics.
- Subscription.
- Team management.
- Appearance & Language settings.
- Privacy & Legal inside AppShell.

هذه ليست blockers لـPR #35، لكنها blockers لتجربة SaaS كاملة وجاهزة للبيع.
