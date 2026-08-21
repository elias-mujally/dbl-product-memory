# المشاكل التي واجهناها وكيف حُلّت

هذا الملف ليس قائمة bugs فقط. الهدف حفظ الدروس حتى لا نعيد نفس المتاهة مستقبلًا.

## 1. Cloud Run pnpm frozen install failure

**المشكلة:**

المشروع خلط إعدادات pnpm 10 مع pnpm 11، وDocker لم يكن ينسخ `pnpm-workspace.yaml` قبل install. ظهرت blocked lifecycle scripts.

**الحل:**

- اعتماد `allowBuilds` الخاص بـpnpm 11.
- مراجعة packages/version واحدة واحدة.
- السماح لـ`sharp` و`unrs-resolver` فقط.
- رفض scripts غير الضرورية لـ`@google/genai` و`protobufjs`.
- إزالة CI bypasses.
- نسخ policy إلى Docker install stages.

**الدرس:**

لا تعطل script security عالميًا لحل build. راجع dependencies بدقة واجعل السياسة committed.

---

## 2. WhatsApp webhook verification لا يعني WABA subscription

**المشكلة:**

Callback verification نجح، لكن التطبيق لم يكن subscribed فعليًا إلى WABA/messages.

**الحل:**

توثيق الفرق بين:

- webhook challenge verification
- app subscription to WABA
- selected webhook fields

**الدرس:**

نجاح GET challenge لا يثبت أن inbound messages ستصل.

---

## 3. Registration/dashboard server error بسبب schema ناقصة

**المشكلة:**

Production PostgREST أعاد `PGRST205` لأن `ai_workspace_settings` لم تكن موجودة، فتحولت إلى server-render exception.

**الحل:**

- fallback محدود فقط لـPGRST205 على optional AI settings.
- استمرار fail-closed لبقية الأخطاء.
- hotfix/reconciliation migrations لاحقًا.

**الدرس:**

Optional relation يجب ألا تسقط الصفحة كلها، لكن لا يجوز ابتلاع permission أو unexpected errors.

---

## 4. Redirects إلى `0.0.0.0:8080` خلف Cloud Run

**المشكلة:**

Next.js بنى origin من container listener بدل public hostname.

**الحل:**

- forwarded-header origin helper.
- احترام `APP_BASE_URL` عند ضبطه.
- tests تمنع العودة إلى `requestUrl.origin` الداخلي.

**الدرس:**

خلف proxies لا تثق بـinternal request origin لبناء callback URLs.

---

## 5. Dashboard crash من AI env numeric values غير صالحة

**المشكلة:**

`GEMINI_TIMEOUT_MS` أو `GEMINI_MAX_OUTPUT_TOKENS` غير الصالحة كانت ترمي exception خارج resilience boundary.

**الحل:**

Safe defaults + structured safe warnings.

**الدرس:**

Optional configuration يجب أن تُحلل دفاعيًا.

---

## 6. Production Supabase schema drift

**المشكلة:**

تاريخ migrations في production اختلف عن repository. Objects وRPCs وconstraints مفقودة، ولا يمكن blind `db push`.

**الحل:**

- read-only introspection.
- forward-only reconciliation migration.
- isolated production-like Supabase project ببيانات اصطناعية.
- تطبيق reconciliation ثم automatic replies migration بالترتيب.
- عدم تعديل التاريخ القديم بالقوة.

**الدرس:**

عند schema drift: reconcile forward، لا تزيف migration history.

---

## 7. Vertex AI authentication على Vercel

**المشكلة:**

Application Default Credentials تعمل جيدًا على Google-hosted environments، لكنها ليست موجودة تلقائيًا على Vercel.

**الحل:**

دعم `GOOGLE_SERVICE_ACCOUNT_JSON` server-only مع الإبقاء على ADC و`GOOGLE_APPLICATION_CREDENTIALS`.

**الدرس:**

مزود واحد قد يحتاج auth paths مختلفة حسب hosting environment.

---

## 8. Fake WhatsApp provider/credential store قد يتسرب إلى hosted runtime

**المشكلة:**

الاعتماد على marker واحد مثل `APP_ENV=production` غير كافٍ.

**الحل:**

Fail-closed عند أي hosted/production marker:

- NODE_ENV
- APP_ENV
- VERCEL_ENV
- Vercel runtime
- Cloud Run K_SERVICE

والـfake store يحتاج explicit opt-in محلي.

**الدرس:**

Fake adapters يجب أن تكون مستحيلة الاختيار في hosted environments، لا مجرد “غير مستحسنة”.

---

## 9. WhatsApp customer secrets داخل نفس project

**المشكلة:**

استخدام project واحد للـVertex AI ولـWhatsApp customer credentials وسّع IAM boundary.

**الحل:**

`WHATSAPP_SECRET_MANAGER_PROJECT` مستقل، مع منع production fallback إلى application project.

**الدرس:**

Customer credentials تستحق boundary منفصلًا وleast privilege.

---

## 10. Legacy `public.reviews` table غير مؤمنة

**المشكلة:**

جدول قديم خارج migration history موجود في production.

**الحل:**

Migration guarded:

- enable + FORCE RLS
- revoke direct privileges
- no policies
- لا إنشاء للجدول في clean environments

**الدرس:**

Legacy objects تحتاج remediation دون إدخال feature غير موجودة في repository.

---

## 11. Embedded Signup كان يظهر “تم الإلغاء” لكل الأخطاء

**المشكلة:**

CANCEL وERROR وmissing code كلها تحولت إلى `provider_cancelled`.

**الحل:**

تصنيف منفصل:

- user_cancelled
- provider_error
- authorization_incomplete
- asset_event_incomplete
- connection_failed

مع privacy-safe diagnostics.

**الدرس:**

لا تحول كل failure إلى “المستخدم ألغى”. ذلك يقتل التشخيص والثقة.

---

## 12. `FB.login()` بعد await فقد user activation

**المشكلة:**

الـclick handler انتظر server action ثم استدعى `FB.login`، ما يرجح فقدان transient user activation.

**الحل:**

- prepare secure state قبل click.
- disable connect حتى preparation.
- synchronous `FB.login` مباشرة من click.

**الدرس:**

Popup/auth flows يجب أن تبدأ داخل user gesture stack.

---

## 13. SDK init timing اختلف عن Meta sample

**المشكلة:**

`FB.init()` كان يحدث لحظة click بدل readiness phase.

**الحل:**

- Next Script `onReady`.
- pre-init قبل enable button.
- `xfbml:true`.
- click يحتوي `FB.login` فقط.

---

## 14. Config ID القديم كان incomplete

**المشكلة:**

التكوين المنتهي بـ3161 كان اسمه يوحي بأنه WhatsApp Embedded Signup، لكن editor أظهر Login Variation incomplete، وعاد accessToken/signedRequest بدل authorization code.

**التحقق:**

صفحة standalone بلا Next.js/React/Supabase أعادت نفس ordinary login behavior.

**الحل:**

إنشاء config جديد صحيح من WhatsApp Embedded Signup template، ينتهي بـ1674.

**الدرس:**

اسم configuration ليس دليلًا على underlying variation.

---

## 15. Authorization code عاد لكن FINISH event لم يظهر

**المشكلة:**

وصل code، لكن لم تصل asset metadata/WA_EMBEDDED_SIGNUP المقبولة.

**التحقيق:**

- listener timing صحيح.
- origin allowlist نوقش.
- حدث postMessage واحد ظهر URL-encoded وغير JSON، ولم يكن WA event.
- readiness/Tech Provider qualification ما زالت ناقصة.

**الحالة:**

متوقف خارجيًا على Business Verification/Tech Provider/App Review، مع ضرورة إعادة الاختبار لاحقًا.

---

## 16. الصفحات القانونية ادعت عمليات غير موجودة

**المشكلة:**

أول نسخة من legal pages تحدثت عن deletion references/status/automation وTerms غير مكتملة وكأنها operational.

**الحل:**

- manual review فقط.
- لا ticket/status promises.
- signup acknowledgment غير تعاقدي.
- retention wording محافظ.
- disclosure لـlocalStorage.
- إزالة “improve service” غير المدعومة.

**الدرس:**

الوثيقة القانونية يجب أن تصف الواقع الحالي، لا roadmap متنكّرًا في بدلة قانونية.

---

## 17. Canonical domain كان يشير إلى Cloud Run/dblab.site

**المشكلة:**

التطبيق الفعلي على `dbl-employee-ai.vercel.app`، بينما docs/canonicals اقترحت origin آخر.

**الحل:**

- `APP_BASE_URL` على Vercel.
- policy URLs على Vercel product domain.
- `dblab.site/contact.html` فقط كـexternal contact.

---

## 18. Locale authority كانت متعددة ومتناقضة

**المشكلة:**

onboarding metadata، business locale، cookie، URL، browser كلها أثرت بلا authority واضحة.

**الحل:**

Locale provenance وترتيب:

1. ui_locale
2. onboarding_ui_locale
3. cookie
4. legacy business fallback
5. browser suggestion على locale-ready routes فقط
6. English

**الدرس:**

لا تمرر locale النهائي فقط؛ احتفظ بمصدر القرار.

---

## 19. Browser Arabic كان يقلب صفحات إنجليزية إلى RTL

**المشكلة:**

Auto-detection فعّل Arabic root على routes لم تُترجم بعد.

**الحل:**

Route readiness policy. Browser suggestion لا يتفعل إلا للمسارات المكتملة.

---

## 20. Auth native validation بلغة المتصفح

**المشكلة:**

واجهة عربية مع browser English عرضت validation messages إنجليزية.

**الحل:**

- `noValidate`
- localized client validation
- aria-invalid/describedby
- focus first invalid field
- server validation محفوظة

---

## 21. Mobile drawer accessibility ناقصة

**المشكلة:**

Dialog semantics بلا focus trap/restoration/inert background.

**الحل:**

- focus entry/trap
- Escape
- restore trigger
- inert background
- scroll lock
- interaction tests

---

## 22. Large bilingual catalogs دخلت client bundle

**المشكلة:**

Protected client error boundary استورد catalog كاملًا.

**الحل:**

Client-safe small error copy boundary، وبقاء product catalogs server-only.

---

## 23. المستخدم لم يرَ التغييرات بسبب discoverability/cache confusion

**المشكلة:**

Knowledge page موجودة على `/knowledge` لكن navigation ركز على `/settings/knowledge`. كذلك tabs/immutable deployments القديمة سببت التباسًا.

**الحل:**

- Knowledge primary nav.
- Knowledge settings distinct.
- active state صحيح.
- production alias هو المرجع.

---

## 24. Knowledge UI كشفت نموذج قاعدة البيانات

**المشكلة:**

Source Type، Priority، كل fieldsets في نموذج واحد.

**الحل:**

Knowledge Hub task-oriented:

- Business
- Products/services
- Policies
- FAQs
- Documents
- Other knowledge

مع forms متخصصة وإخفاء technical fields.

---

## 25. Readiness جعلت PDF شرطًا للوصول إلى 100%

**المشكلة:**

Documents وزنها 15% رغم أنها ليست لازمة لكل نشاط.

**الحل:**

Readiness = 30/30/20/20، Documents optional enrichment.

---

## 26. Edit UX كان يمكن أن يمحو بيانات مخفية

**المشكلة:**

Omitted description أصبح null، policy title استُبدل، pricing mode التبس.

**الحل:**

- distinguish omitted vs intentional clear.
- merge existing values.
- preserve title/description/priority/language/source_type/metadata.
- pricing mode داخل metadata مع legacy inference.
- round-trip tests.

---

## 27. عبارة “sent for review” أوحت أن DBL يراجع العلامة

**المشكلة:**

العميل قد يظن أن DBL يتحكم في محتواه.

**الحل:**

- Owner/Admin: Make available.
- Agent: submit to owner/admin.
- Viewer read-only.
- Review queue داخل workspace.
- copy صريحة أن DBL لا يراجع المحتوى.

---

## 28. Create-and-publish لم يكن idempotent

**المشكلة:**

Double click أو transport retry ينشئ duplicates.

**الحل الجاري في PR #35:**

- migration forward-only.
- replay ledger table.
- unique actor/workspace/key.
- fingerprint.
- SECURITY DEFINER RPC.
- client pending lock كـUX فقط.
- partial failure recovery.

**المشكلة الأخيرة:**

“Save and add another” كان يحتفظ بالمفتاح السابق. تم إصلاحه محليًا ولم يُدفع بعد بسبب Codex external command limit.