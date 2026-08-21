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

---

## 15. Contacts: WhatsApp-first, Channel-ready

**القرار:**

Contacts تُبنى كهوية عميل عامة قابلة لاستيعاب قنوات مستقبلية، بينما يبقى WhatsApp القناة الوحيدة المدعومة فعليًا الآن.

**السبب:**

لا نريد ربط Contact نهائيًا بواتساب، ولا نريد في المقابل تنفيذ Instagram/Facebook أو تعميم provider infrastructure مبكرًا.

**النتيجة:**

- Contact مستقل عن provider.
- Channel Identity منفصلة.
- WhatsApp-specific infrastructure يبقى WhatsApp-specific حاليًا.

---

## 16. Contact ID هو UUID داخلي، وليس رقم هاتف أو Provider ID

**القرار:**

المعرف الأساسي للـContact هو UUID داخلي من DBL.

**ممنوع كـprimary identity:**

- phone number
- email
- WhatsApp user ID
- Meta provider ID

**السبب:**

هذه القيم يمكن أن تتغير أو تتعدد، بينما Contact يجب أن يبقى كيانًا مستقرًا طويل العمر.

---

## 17. WhatsApp account scope للـContact Identity يستخدم `phone_number_id`

**القرار:**

في التصميم الحالي، uniqueness لهوية WhatsApp تعتمد على snapshot من receiving `phone_number_id`، لا على `whatsapp_connection_id` وحده.

**السبب:**

التدقيق كشف أن connection UUID قد يُعاد استخدامه بينما تتغير provider-account metadata عند إعادة الربط، لذلك ليس immutable account identifier كافيًا.

**الـinvariant:**

```text
workspace_id + channel + channel_account_external_id + external_user_id
= one Channel Identity
```

---

## 18. Legacy compatibility عبر `contact_channel_identities.legacy_customer_id`

**القرار:**

لا نضيف `contact_id` إلى `customers` في PR 1، ولا نجعل `contacts` يحمل `legacy_customer_id`.

الربط المؤقت يكون عبر:

`contact_channel_identities.legacy_customer_id`

**السبب:**

- `customers` يبقى untouched.
- WhatsApp customer يربط بهويته الصحيحة أولًا، ثم Contact.
- لا dual source of truth.
- لا mapping table ثالث غير ضروري.
- إزالة legacy layer مستقبلًا أوضح.

---

## 19. Contacts PR 1 لا يلمس Conversations أو Messages أو Ingestion

**القرار:**

في PR 1:

- `conversations`: unchanged
- `messages`: unchanged
- WhatsApp ingestion behavior: unchanged
- outbound routing: unchanged
- AI behavior: unchanged
- لا `/contacts` UI

**السبب:**

PR 1 يجب أن يكون additive foundation فقط. ربط Conversations الآن سيخلق dual-write ومخاطر drift قبل إثبات الهوية الجديدة.

---

## 20. Contacts RLS في MVP: جميع أعضاء Workspace النشطين يقرؤون، ولا Browser Writes

**القرار:**

Owner/Admin/Agent/Viewer يستطيعون قراءة Contacts داخل Workspace النشط نفسه.

- لا authenticated browser INSERT.
- لا authenticated browser UPDATE.
- لا authenticated browser DELETE.
- provider/ingestion writes تبقى server/service-controlled.

**السبب:**

Contacts MVP read-only ويتوافق مع نموذج customer/conversation visibility الحالي، مع الحفاظ على tenant isolation.

---

## 21. لا Merge ولا Delete للـContacts في MVP

**القرار:**

- لا automatic merge.
- لا manual merge.
- لا Contact deletion.
- Disconnect لا يحذف Contact/Identity/history.

**السبب:**

الدمج اعتمادًا على الاسم أو identifiers غير الموثوقة خطر، وحذف integration ليس طلب حذف تاريخ العميل.

---

## 22. Provider Name له أولوية محدودة فقط

**القرار:**

Provider display name يمكنه تحديث Contact name فقط إذا:

- Contact name فارغ؛ أو
- Contact name ما يزال مساويًا للقيمة السابقة القادمة من Provider.

أي manual Contact name مستقبلًا له أولوية ولا يُمسح بتحديث provider.

**السبب:**

منع provider refresh من الكتابة فوق بيانات أعلى ثقة دون الحاجة لنظام provenance معقد في MVP.

---

## 23. Production Contacts Preflight هو Go/No-Go gate

**القرار:**

لا يبدأ Contacts PR 1 implementation قبل read-only production preflight يثبت:

- customer count معروف؛
- zero workspace/connection mismatch؛
- zero customers على connection بلا `phone_number_id`؛
- zero proposed identity collisions؛
- لا anomaly تحتاج remediation.

**إذا ظهرت anomaly:**

STOP. لا auto-fix ولا auto-merge ولا normalization صامت.

**السبب:**

المعمارية يجب أن تتكيف مع البيانات الحقيقية، لا أن تخمن أثناء migration.

---

## 24. PR 2 يجب أن يغلق فجوة PR 1 قبل تفعيل Contacts UI

**القرار:**

PR 2 يتبع PR 1 مباشرة قدر الإمكان ويضيف dual-write/catch-up للـContacts الناتجة من WhatsApp ingestion.

لا يتم تفعيل `/contacts` قبل إثبات:

- runtime Contact/Identity creation/linking؛
- catch-up backfill للعملاء الذين ظهروا بعد snapshot الخاص بـPR 1؛
- اكتمال التغطية بدون duplicates.

**السبب:**

PR 1 لا يغيّر ingestion، وبالتالي توجد فجوة زمنية مقصودة وآمنة فقط طالما لا تعتمد UI عليها.

---

## 25. تعقيد Backfill يعتمد على حجم Production الحقيقي

**القرار:**

لا نبني batching/monitoring infrastructure مسبقًا.

- حجم صغير/متوسط → migration بسيطة.
- حجم/مخاطر lock/WAL كبيرة → redesign rollout قبل التنفيذ.

**السبب:**

الاحتياط يجب أن يكون متناسبًا مع البيانات الفعلية، لا مع افتراضات نظرية.
