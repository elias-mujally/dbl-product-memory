# الحالة الحالية

آخر تحديث: **2026-08-12**

## ملخص سريع

DBL Employee AI تطبيق SaaS متعدد المستأجرين بواجهة عربية/إنجليزية، Knowledge Hub منظم، AI grounded على معرفة معتمدة، وWhatsApp/Meta Embedded Signup جاهز تقنيًا للاختبار الإداري في Production بينما تبقى موافقات Meta التجارية الخارجية مسارًا منفصلًا.

التركيز الحالي هو **Contacts** قبل Analytics.

تم إغلاق **Contacts PR 1 — Foundation** بالكامل، وتم الآن دمج **PR #38 — WhatsApp Account-Scope Lifecycle Guard** كمتطلب سابق لـContacts PR 2.

الحالة الحالية:

> **PR #38 merged. Post-merge Production verification is the immediate next step. Contacts PR 2 has not started yet.**

---

## الإنتاج الحالي

- Production: `https://dbl-employee-ai.vercel.app`
- آخر main مؤكد قبل دمج #38: `64f25ae666d9cd2b62cd1caa37f9f9c2d2f84ae3`
- PR #38 reviewed head: `1325412e13fbf8f8925127ec231d744f2a1d0caa`
- PR #38 squash SHA: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- PR #38: **MERGED via squash**
- Production verification لـ#38: **pending**

مرجع الحدث:

`events/2026-08-12-pr38-whatsapp-account-scope-guard-merged.md`

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
- `events/2026-08-12-pr38-whatsapp-account-scope-guard-merged.md`

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

- legacy compatibility عبر `contact_channel_identities.legacy_customer_id`.
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

---

## PR #38 — WhatsApp Account-Scope Lifecycle Guard — MERGED / VERIFY NEXT

### لماذا كان مطلوبًا

أثناء تخطيط Contacts PR 2 اكتشف audit أن Embedded Signup كان يستطيع إعادة استخدام `whatsapp_connections` وتغيير receiving `phone_number_id` in-place، بينما Contacts Foundation تعتبر هذا الرقم account scope تاريخيًا لهوية WhatsApp.

تم اعتماد السياسة:

> بمجرد امتلاك الاتصال customer history أو canonical Contact identity history، لا يجوز استبدال `phone_number_id` الخاص به in-place. إعادة ربط الرقم نفسه مسموحة. رقم مختلف يحتاج مستقبلًا account scope/connection منفصلًا، بدون automatic Contact merge.

### السلوك النهائي

- new connection: unchanged.
- same `phone_number_id`: allowed.
- different `phone_number_id` without history: existing behavior allowed.
- different number with customer أو canonical identity history: transactionally rejected.
- Contacts/identities/customers/conversations/messages/history محفوظة.
- multi-connection غير مدعوم بعد، لذلك لا يتم وعد المستخدم بمسار غير موجود.

### أهم المشاكل التي اكتشفت أثناء المراجعات وأغلقت

- stale credential predecessor race.
- service-level concurrency test gap.
- misleading separate-connection recovery copy.
- ambiguous committed RPC response deleting active credential.
- non-durable `NOT_COMMITTED` classification.

الحل النهائي لـ`NOT_COMMITTED`:

- reconciliation يقفل signup session.
- confirmed non-commit يحول session من `completing` إلى terminal `failed`.
- reason: `whatsapp_completion_reconciled_not_committed`.
- بعد commit هذا fence فقط يسمح candidate cleanup.
- delayed/retried completion لا يستطيع العودة للـcommit لأنه يحتاج `status='completing'`.

### Final review

Reviewed head:

`1325412e13fbf8f8925127ec231d744f2a1d0caa`

نتيجة المراجعة النهائية:

- High: **0**
- Medium: **0**
- Production risk: **Low**
- T1–T7 concurrency: passed
- service-contract tests: passed
- Vitest: 464 passed
- pgTAP: 466 passed
- Supabase reset / Contacts upgrade / authenticated E2E / Vercel Preview: passed

### Merge

- squash SHA: `57052f665150849c1040bc173a8aef3bb9b02ab8`
- merge confirmed by GitHub.

### Remaining Low operational debt

- reconciled ACTIVE after lost response may leave predecessor credential for manual protected cleanup.
- credential cleanup is observable but no durable automatic retry queue exists yet.
- Meta-side activation may precede DBL rejection while DBL state remains fail-closed.

### Immediate next step

**Post-merge Production verification لـPR #38.**

لا تعتبر #38 production-closed حتى نثبت:

- migrations applied exactly once/in order;
- Production deployment READY on squash SHA;
- no migration/RPC/runtime/5xx cluster;
- existing WhatsApp history/credentials remain intact;
- role restrictions and account-scope guard remain correct.

---

## Known PR 1 → PR 2 gap

PR 1 deliberately did not change ingestion.

Therefore a customer created after PR 1 migration and before PR 2 integration may temporarily lack canonical Contact/identity.

آخر verified count قبل #38:

**0 unmapped post-migration customers**

The gap remains safe only because Contacts runtime/UI is disabled.

Operational rule:

- بعد إغلاق Production verification لـ#38، نعود مباشرة إلى PR 2.
- PR 2 must add atomic create/link dual-write behavior.
- PR 2 must include idempotent catch-up backfill.
- PR 2 must close the ingestion race before final catch-up validation.
- `/contacts` must remain disabled until PR 2 proves complete canonical coverage.

---

## Contacts PR 2 — NEXT AFTER PR #38 PRODUCTION VERIFICATION

Goal:

**WhatsApp Contact Integration**

Expected scope:

- integrate canonical Contact + Channel Identity creation/linking into WhatsApp ingestion;
- preserve existing customer/conversation/message behavior;
- close retries/concurrency duplication risk;
- perform catch-up for customers created after PR 1;
- prove one intended WhatsApp identity maps to one canonical identity;
- preserve provider data precedence rules;
- keep Contacts UI disabled;
- no Instagram/Facebook integration;
- no legacy removal yet.

The PR 2 technical audit already identified database-enforced dual-write as the preferred architecture, but implementation remains blocked until PR #38 is production-verified and Product Memory is updated again.

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

- لا تبدأ Contacts PR 2 قبل Production verification لـ#38.
- لا تفتح `/contacts` قبل PR 2.
- لا تبدأ Analytics قبل تثبيت Contacts identity semantics.
- لا تضف Instagram/Facebook integration.
- لا تعمم `whatsapp_connections` قبل وجود قناة ثانية فعلية.
- لا تعمل Contact merge تلقائيًا.
- لا تحذف `customers` أو customer history.
- لا تعتبر PR #38 production-closed قبل التحقق الإنتاجي.
