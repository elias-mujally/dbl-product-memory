# الحالة الحالية

آخر تحديث: **2026-08-11**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp/Meta Embedded Signup أصبح جاهزًا تقنيًا للاختبار الإداري بعد دمج PR #36 والتحقق الإنتاجي الناجح.

التركيز الحالي هو **Contacts Foundation** قبل Analytics.

تم اعتماد وإنجاز:

- Product architecture لميزة Contacts.
- UX direction للـContacts MVP.
- Technical audit كامل على `main`.
- Technical Specification لـContacts PR 1.
- مراجعة مستقلة من DeepSeek وافقت على التصميم.
- **Production Contacts Preflight قراءة فقط: GO.**

**Contacts PR 1 — Foundation أصبح مصرحًا ببدء تنفيذه.**

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

## Production Contacts Preflight — PASSED

المرجع:

`CONTACTS_PRODUCTION_PREFLIGHT.md`

### النتيجة

**GO**

### Production data observed

- total customers: **1**
- workspaces with customers: **1**
- customer-linked WhatsApp connections: **1**
- connected: **1**

### Blocking anomaly checks

- customer/workspace vs connection/workspace mismatches: **0**
- customers linked to connection missing `phone_number_id`: **0**
- proposed canonical identity collision groups: **0**
- WhatsApp user ID null/empty/whitespace/length anomalies: **0**
- normalized identity collisions: **0**
- duplicate legacy customer mapping violations: **0**
- missing `profile_name`: **0**
- missing `phone_number_normalized`: **0**

### Migration sizing

**Tiny**.

المعتمد حاليًا:

- straightforward transactional backfill مناسب.
- batching غير مطلوب.
- online/concurrent index strategy غير مطلوب.
- monitoring infrastructure خاص بالـbackfill غير مطلوب بهذا الحجم.

إذا تغير حجم Production ماديًا قبل التنفيذ يجب إعادة تقييم هذا الحكم.

### PR 1 authorization

بما أن preflight عاد بـzero blockers:

> **Contacts PR 1 — Foundation may begin using the approved Technical Specification unchanged unless implementation reveals a new concrete blocker.**

## PR sequencing المعتمد

### PR 1 — Contacts Foundation — NEXT

- schema + RLS + backfill + tests فقط.
- additive / forward-only.
- لا runtime ingestion change.
- لا UI.

### PR 2 — WhatsApp Contact Integration

يأتي مباشرة بعد PR 1 قدر الإمكان:

- dual-write / create-link Contact + identity من ingestion.
- catch-up backfill للعملاء الذين ظهروا بين PR 1 وPR 2.
- duplicate/idempotency/concurrency coverage.
- إغلاق ingestion race قبل final catch-up validation.

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

تعقيد backfill يتحدد من Production data الفعلية.

Preflight الحالي يصنف الحجم **Tiny**، لذلك لا نضيف batching/monitoring معقدًا بلا حاجة.

## Independent review

### Codex

- audit read-only كامل.
- PR 1 risk قبل preflight: **4/10 — Low-to-medium**.
- Production preflight: **GO**.
- implementation authorization: yes.

### DeepSeek

- وافق على architecture/technical audit.
- أكد preflight كـGo/No-Go.
- أوصى بإبقاء PR 2 قريبًا جدًا من PR 1.
- أوصى بمراقبة migration proportional to actual volume.

تم اعتماد هذه التوصيات، ونتيجة Production الحالية لا تستدعي batching أو migration-monitoring infrastructure إضافية.

## الخطوة التالية حرفيًا

1. إعطاء Codex مهمة تنفيذ **Contacts PR 1 — Foundation**.
2. إنشاء Draft PR فقط، بلا دمج أو نشر يدوي.
3. الالتزام بحدود `CONTACTS_TECHNICAL_SPEC.md`.
4. تشغيل migration-upgrade tests وpgTAP/RLS/regression suites.
5. self-review نقدي.
6. إصلاح أي High/Medium findings.
7. independent final review قبل squash merge.
8. بعد PR 1، الانتقال بسرعة إلى PR 2 لإغلاق dual-write/catch-up gap قبل أي Contacts UI.

## لا تفعل الآن

- لا تفتح `/contacts` UI داخل PR 1.
- لا تعدل WhatsApp ingestion ضمن PR 1.
- لا تعدل `conversations` أو `messages` ضمن PR 1.
- لا تضف Instagram/Facebook integration.
- لا تعمم `whatsapp_connections` قبل وجود قناة ثانية فعلية.
- لا تعمل merge تلقائي للعملاء.
- لا تحذف `customers` أو Conversations history.
- لا توسع PR 1 إلى CRM features أو Analytics.
