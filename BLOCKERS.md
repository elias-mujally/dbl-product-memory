# العوائق الحالية

آخر تحديث: 2026-08-08

## 1. Meta Business Verification

**الحالة:** Pending / In progress لأكثر من أسبوع.

**الأثر:**

يمنع أو يؤخر:

- إكمال Independent Tech Provider qualification.
- App Review.
- Advanced Access.
- Customer production onboarding.

**المطلوب:**

انتظار Meta، وعدم الضغط على resend إلا عند وجود دليل أن الطلب عالق أو طلب Meta ذلك. إذا تجاوز الانتظار مدة طويلة إضافية بلا Required Actions، فالتصعيد إلى Meta Business Support يصبح منطقيًا.

---

## 2. Tech Provider qualification

**الحالة:** بدأ، لكنه متوقف على Business Verification.

**لاحقًا بعد التحقق:**

- مراجعة app settings.
- screencasts.
- App Review materials.
- permission justifications.

---

## 3. WhatsApp Advanced Access

**غير مكتمل:**

- `whatsapp_business_management`
- `whatsapp_business_messaging`

الحالة السابقة: Ready for testing فقط.

**الأثر:**

Admin/developer testing قد يعمل ضمن حدود، لكن external customer production onboarding غير جاهز.

---

## 4. Embedded Signup FINISH/asset event

**الحالة:** غير محسومة نهائيًا بعد config الجديد.

تم إثبات:

- authorization code يعود مع config 1674.
- config القديم كان خاطئًا.

لم يُثبت بعد end-to-end customer asset grant مع WABA/phone metadata في الإنتاج.

**بعد Meta approval:**

إعادة اختبار المسار الكامل حتى Complete/FINISH، ثم backend exchange والتفعيل.

---

## 5. Vercel Meta/WhatsApp environment audit

Google Cloud وSecret Manager وحساب الخدمة والصلاحيات جاهزة مسبقًا.

المتبقي هو التأكد من متغيرات الإنتاج/Preview في Vercel، خصوصًا:

- `META_EMBEDDED_SIGNUP_CONFIG_ID`
- `META_APP_ID`
- `META_APP_SECRET`
- `WHATSAPP_PROVIDER=meta`
- `WHATSAPP_GRAPH_API_VERSION=v25.0`
- `WHATSAPP_CREDENTIAL_STORE=gcp_secret_manager`
- `WHATSAPP_SECRET_MANAGER_PROJECT`
- `WHATSAPP_VERIFY_TOKEN`

هذا تدقيق إعدادات، وليس سببًا لإعادة بناء Google Cloud.

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

هذه ليست blockers للإنتاج الحالي، لكنها أجزاء مهمة للوصول إلى تجربة SaaS أوسع وجاهزة للبيع.

---

## مغلق: PR #35 / Codex limit

العائق السابق الخاص بحد Codex وPR #35 أُغلق بالكامل:

- عاد Codex.
- تم دفع الإصلاحات.
- اجتاز PR #35 المراجعات النهائية دون High/Medium.
- تم squash merge إلى `main` عند SHA `b14c1fe2e144cb58cd0618501abe3f10c5a88494`.
- migration الإنتاجية طُبقت بنجاح مرة واحدة.
- Vercel production أصبح READY.

لا يُعامل PR #35 كعائق حالي بعد الآن.
