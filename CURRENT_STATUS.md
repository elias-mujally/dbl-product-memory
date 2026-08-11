# الحالة الحالية

آخر تحديث: **2026-08-11**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp/Meta Embedded Signup أصبح جاهزًا تقنيًا للاختبار الإداري بعد دمج PR #36 والتحقق الإنتاجي الناجح.

التركيز الحالي انتقل إلى **Contacts Foundation** قبل Analytics.

تم اعتماد:

- Product architecture لميزة Contacts.
- UX direction للـContacts MVP.
- Technical audit كامل على `main`.
- Technical Specification لـContacts PR 1.
- مراجعة مستقلة من DeepSeek وافقت على التصميم مع شروط preflight/rollout.

**التنفيذ لم يبدأ بعد.**

الخطوة التالية هي **Read-only Production Contacts Preflight** كـGo/No-Go gate قبل أي migration.

## الإنتاج الحالي

- Production: `https://dbl-employee-ai.vercel.app`
- `main`: `d4d5dbfa988dc2e653f0164193fb9df5a4aba5f2`
- آخر PR مدمج: **#36**
- Meta administrator testing readiness: verified in Production.
- Meta commercial approvals الخارجية ما زالت مسارًا منفصلًا وغير مكتمل.

## حالة Contacts الحالية

### Architectural direction

**WhatsApp-first, Channel-ready**.

- WhatsApp هو القناة الوحيدة المدعومة الآن.
- Contact لا يجب أن يكون مساويًا دائمًا لـWhatsApp customer.
- Future channels تبقى ممكنة بدون تنفيذها الآن.
- `whatsapp_connections` وبنية provider تبقى WhatsApp-specific حاليًا ولا يتم تعميمها مبكرًا.

المرجع:

`CONTACTS_ARCHITECTURE.md`

### Technical Specification

المرجع:

`CONTACTS_TECHNICAL_SPEC.md`

تم اعتماد تصميم PR 1 التالي:

- جدول جديد `contacts`.
- جدول جديد `contact_channel_identities`.
- Contact primary ID = DBL internal UUID.
- WhatsApp account scope الحالي = receiving `phone_number_id` snapshot.
- uniqueness المنطقية:

```text
workspace_id + channel + channel_account_external_id + external_user_id
= one Channel Identity
```

- legacy compatibility عبر:

`contact_channel_identities.legacy_customer_id`

- `customers` يبقى موجودًا وغير معدل في PR 1.
- `conversations` لا تتغير في PR 1.
- `messages` لا تتغير في PR 1.
- WhatsApp ingestion behavior لا يتغير في PR 1.
- outbound routing لا يتغير في PR 1.
- AI behavior لا يتغير في PR 1.
- `/contacts` UI لا تُفتح في PR 1.

### Permissions

Contacts MVP read-only:

- Owner: read
- Admin: read
- Agent: read
- Viewer: read

داخل Workspace النشط فقط.

Normal browser writes ممنوعة.

Provider/ingestion writes تبقى server/service-controlled.

### Merge / Delete / Disconnect

في MVP:

- لا automatic merge.
- لا manual merge.
- لا Contact deletion.
- WhatsApp disconnect لا يحذف Contact أو ChannelIdentity أو customer history أو conversations/messages.

### Provider name precedence

Provider display name يستطيع تحديث Contact name فقط إذا كان Contact name:

- فارغًا؛ أو
- ما يزال مساويًا للقيمة السابقة القادمة من Provider.

أي manual name مستقبلي له أولوية.

## Production Preflight — الخطوة التالية

لا يبدأ PR 1 implementation قبل فحص Production قراءة فقط يثبت:

1. customer row count وحجم migration المتوقع.
2. zero customer workspace / WhatsApp connection workspace mismatches.
3. zero customers attached to connections without `phone_number_id`.
4. zero proposed identity collisions بعد normalization المطلوبة.
5. عدم وجود anomaly تحتاج remediation خاصة.

إذا ظهر أي anomaly:

**STOP.**

لا auto-fix، لا auto-merge، ولا normalization صامت.

يجب تصميم remediation مستقلة أولًا.

## PR sequencing المعتمد

### PR 1 — Contacts Foundation

- schema + RLS + backfill + tests فقط.
- additive / forward-only.
- لا runtime ingestion change.
- لا UI.

### PR 2 — WhatsApp Contact Integration

يأتي مباشرة بعد PR 1 قدر الإمكان:

- dual-write / create-link Contact + identity من ingestion.
- catch-up backfill للعملاء الذين ظهروا بين PR 1 وPR 2.
- duplicate/idempotency/concurrency coverage.

### PR 3 — Contacts MVP UI

فقط بعد إثبات اكتمال PR 2:

- `/contacts`
- `/contacts/[contactId]`
- search/pagination
- Arabic/English
- desktop/mobile
- read-only customer-memory experience

### PR 4 — Legacy reduction later

بعد استقرار المسار الجديد في Production، نراجع بشكل مستقل دور `customers` القديم.

لا يوجد قرار مسبق بحذفه.

## Performance rule

تعقيد backfill يتحدد من Production preflight:

- data صغيرة/متوسطة → migration بسيطة.
- volume أو lock/WAL risk ملحوظ → إعادة تصميم rollout/batching قبل التنفيذ.

لا نبني مراقبة أو batching معقدة من باب الاحتياط النظري فقط.

## Independent review

### Codex

- audit read-only كامل.
- PR 1 risk: **4/10 — Low-to-medium**.
- recommendation: implementation conditionally ready after production preflight.

### DeepSeek

- وافق على architecture/technical audit.
- أكد ضرورة حسم open decisions.
- أكد preflight كـGo/No-Go.
- أوصى بإبقاء PR 2 قريبًا جدًا من PR 1.
- أوصى بمراقبة migration proportional to actual volume.

تم اعتماد هذه التوصيات.

## الخطوة التالية حرفيًا

1. إعطاء Codex مهمة **Production Contacts Preflight — Read Only**.
2. عدم تعديل أي file/schema/config/data أثناءها.
3. مراجعة نتائج counts/anomalies.
4. إذا zero blockers → إعطاء الضوء الأخضر لتنفيذ **Contacts PR 1 Foundation**.
5. إذا anomaly → توقف وتصميم remediation قبل أي migration.

## لا تفعل الآن

- لا تبدأ Contacts migration قبل preflight.
- لا تفتح `/contacts` UI.
- لا تعدل WhatsApp ingestion ضمن PR 1.
- لا تضف Instagram/Facebook integration.
- لا تعمم `whatsapp_connections` قبل وجود قناة ثانية فعلية.
- لا تعمل merge تلقائي للعملاء.
- لا تحذف `customers` أو Conversations history.
