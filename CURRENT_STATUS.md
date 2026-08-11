# الحالة الحالية

آخر تحديث: **2026-08-11**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp/Meta Embedded Signup جاهز تقنيًا للاختبار الإداري في Production بينما تبقى موافقات Meta التجارية الخارجية مسارًا منفصلًا.

التركيز الحالي هو **Contacts** قبل Analytics.

تم الآن إغلاق **Contacts PR 1 — Foundation** بالكامل بعد الدمج والتحقق الإنتاجي.

الخطوة التالية المعتمدة:

> **Contacts PR 2 — WhatsApp Contact Integration**

PR 2 لم يبدأ بعد.

---

## الإنتاج الحالي

- Production: `https://dbl-employee-ai.vercel.app`
- `main`: `64f25ae666d9cd2b62cd1caa37f9f9c2d2f84ae3`
- آخر PR مدمج: **#37 — Contacts Foundation**
- Vercel deployment: `dpl_EKRnti8TQDxjX5xiF8Cn6CGm3LMu`
- Vercel status: **READY**

---

## Contacts — الاتجاه المعتمد

**WhatsApp-first, Channel-ready**.

- WhatsApp هو القناة الوحيدة المدعومة الآن.
- Contact كيان مستقل عن WhatsApp customer على المدى الطويل.
- Future channels تبقى ممكنة دون تنفيذها الآن.
- WhatsApp provider infrastructure تبقى WhatsApp-specific حاليًا ولا يتم تعميمها مبكرًا.

المراجع:

- `CONTACTS_ARCHITECTURE.md`
- `CONTACTS_TECHNICAL_SPEC.md`
- `CONTACTS_PRODUCTION_PREFLIGHT.md`
- `events/2026-08-11-pr37-contacts-foundation-merged.md`

---

## Contacts PR 1 — CLOSED

### Merge

- PR: **#37**
- الحالة: **MERGED via squash**
- reviewed head: `9de793fd0c9d2cfb0fa55f35b85ebbe8ed4f614d`
- squash/main SHA: `64f25ae666d9cd2b62cd1caa37f9f9c2d2f84ae3`

### Migration

Repository source:

`20260811183306_contacts_foundation.sql`

Production execution version:

`20260811195845`

- applied exactly once
- no unrelated migration applied
- no preflight / lock / constraint error
- historical local/remote migration-ledger divergence remains separate operational debt and was not repaired

### New canonical foundation

تم إنشاء:

- `contacts`
- `contact_channel_identities`

القواعد الأساسية:

- Contact primary ID = DBL internal UUID.
- `channel='whatsapp'` فقط حاليًا.
- WhatsApp account scope = receiving `phone_number_id` snapshot.
- uniqueness:

```text
workspace_id + channel + channel_account_external_id + external_user_id
= one Channel Identity
```

- legacy compatibility عبر:

`contact_channel_identities.legacy_customer_id`

- `customers` بقي موجودًا.
- `conversations` لم تتغير.
- `messages` لم تتغير.
- WhatsApp ingestion behavior لم يتغير في PR 1.
- outbound/AI لم يتغيرا.

### Production backfill verification

بعد migration:

- legacy customers: **1**
- canonical Contacts: **1**
- compatibility identities: **1**
- customers بدون identity: **0**
- orphan Contacts: **0**
- orphan identities: **0**
- cross-workspace / relationship mismatches: **0**

Legacy preservation:

- customers: **1**
- conversations: **1**
- messages: **17**

لم يُحذف أي historical customer/conversation/message.

### RLS / privileges

Production verification passed:

- RLS enabled.
- FORCE RLS enabled.
- `anon` denied.
- `authenticated` = workspace-scoped SELECT only.
- browser INSERT/UPDATE/DELETE denied.
- `service_role` = SELECT/INSERT/UPDATE only.
- no service-role DELETE.

### Runtime health

- Vercel Production: READY.
- no runtime error clusters.
- no 5xx cluster.
- no Contacts-related Supabase error entries detected.
- no WhatsApp runtime regression detected.
- no synthetic Production provider traffic was created just for verification.

### Contacts UI remains disabled

- `/contacts` returns 404.
- navigation remains `href: null`.
- no runtime UI depends on canonical Contacts yet.

---

## Known PR 1 → PR 2 gap

PR 1 deliberately did not change ingestion.

Therefore a customer created after PR 1 migration and before PR 2 integration may temporarily lack canonical Contact/identity.

Current verified unmapped post-migration customers:

**0**

The gap remains safe only because Contacts runtime/UI is disabled.

Operational rule:

- PR 2 should follow directly / within a short engineering window.
- PR 2 must add atomic create/link dual-write behavior.
- PR 2 must include idempotent catch-up backfill.
- PR 2 must close the ingestion race before final catch-up validation.
- `/contacts` must remain disabled until PR 2 proves complete canonical coverage.

---

## Contacts PR 2 — NEXT

Goal:

**WhatsApp Contact Integration**

Expected scope to plan/review before implementation:

- integrate canonical Contact + Channel Identity creation/linking into WhatsApp ingestion;
- preserve existing customer/conversation/message behavior;
- close retries/concurrency duplication risk;
- perform catch-up for customers created after PR 1;
- prove one intended WhatsApp identity maps to one canonical identity;
- preserve provider data precedence rules;
- keep Contacts UI disabled;
- no Instagram/Facebook integration;
- no legacy removal yet.

PR 2 must be designed as a separate reviewed task. Do not assume implementation details before a fresh current-main audit.

---

## Later sequencing

### PR 3 — Contacts MVP UI

Only after PR 2 is production-verified:

- `/contacts`
- `/contacts/[contactId]`
- search/pagination
- Arabic/English
- desktop/mobile
- read-only customer-memory UX

### PR 4 — Legacy reduction later

After the canonical path is proven in Production, independently review the future role of `customers`.

No pre-approved requirement exists to delete it.

---

## لا تفعل الآن

- لا تفتح `/contacts` قبل PR 2.
- لا تبدأ Analytics قبل تثبيت Contacts identity semantics.
- لا تضف Instagram/Facebook integration.
- لا تعمم `whatsapp_connections` قبل وجود قناة ثانية فعلية.
- لا تعمل Contact merge تلقائيًا.
- لا تحذف `customers` أو customer history.
- لا تبدأ PR 2 بدون مراجعة current `main` ووضع Technical Plan خاص به.
