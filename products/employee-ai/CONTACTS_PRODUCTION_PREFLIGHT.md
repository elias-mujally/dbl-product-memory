# Contacts Production Preflight

**Date:** 2026-08-11

**Status:** **GO**

This file records the mandatory read-only Production Contacts Preflight that was required before authorizing **Contacts PR 1 — Foundation**.

The preflight was executed against the active DBL production Supabase project and made **no file, schema, configuration, migration, or production-data changes**.

## Production alignment

- Active production Supabase project: confirmed.
- Current application `main`: `d4d5dbfa988dc2e653f0164193fb9df5a4aba5f2`.
- Schema state matches current post-PR #36 application state.
- Latest expected Knowledge idempotency migration is present.
- `contacts` and `contact_channel_identities` do not yet exist, as expected before PR 1.

## Production customer volume

- Total `customers`: **1**
- Workspaces containing customers: **1**
- WhatsApp connections referenced by customers: **1**
- Distribution: one customer in one workspace on one WhatsApp connection.

### Migration size classification

**Tiny**.

At the current production volume, the approved straightforward transactional backfill is appropriate.

No evidence currently justifies:

- batched backfill;
- online/concurrent index creation;
- external migration progress infrastructure;
- complex production monitoring specifically for this migration.

If production volume changes materially before implementation, the preflight must be reconsidered.

## Go/No-Go anomaly checks

### Workspace / connection consistency

Expected invariant:

```text
customers.workspace_id = whatsapp_connections.workspace_id
```

Result:

- mismatch count: **0**

### WhatsApp account scope

Approved Contacts identity design uses the receiving Meta `phone_number_id` as the WhatsApp account-scope snapshot.

Result:

- customers attached to connection with missing/null `phone_number_id`: **0**
- affected connections: **0**

### Proposed canonical identity collisions

Approved future identity tuple:

```text
workspace_id
+ channel = whatsapp
+ channel_account_external_id = phone_number_id
+ external_user_id = whatsapp_user_id
```

Result:

- collision groups: **0**
- excess collision rows: **0**

### WhatsApp user ID quality

Result:

- null: **0**
- empty after trim: **0**
- surrounding whitespace: **0**
- outside current 3–64 character ingestion constraint: **0**
- normalized identity collisions: **0**

No production normalization or remediation is required before PR 1.

### Legacy uniqueness

Result:

- duplicate customer UUID rows: **0**
- duplicate `(whatsapp_connection_id, whatsapp_user_id)` rows/groups: **0**
- candidate `legacy_customer_id` mapping violations: **0**

The approved compatibility mechanism remains viable:

```text
contact_channel_identities.legacy_customer_id
```

### Optional customer profile data

Result:

- missing `profile_name`: **0**
- missing `phone_number_normalized`: **0**

These fields would not have been blockers because the approved specification permits nullable Contact display name / phone-derived identity data where required.

## Customer-linked WhatsApp connection states

Current distribution:

- `connected`: **1**

No disconnected/pending/error/suspended customer-linked connection exists in the current production sample.

The architectural rule remains unchanged: disconnect must preserve Contact, Channel Identity, legacy customer, conversations, messages, and history.

## Existing security posture confirmed

Relevant production expectations were confirmed:

- RLS is enabled on current tenant-owned WhatsApp/customer messaging tables.
- Existing legacy tables do not currently use FORCE RLS; the new Contacts tables are still specified to use RLS + FORCE RLS.
- `anon` has no customer access.
- `authenticated` has workspace-scoped customer read access but no direct customer INSERT/UPDATE/DELETE.
- WhatsApp ingestion is service-controlled through the hardened server-side ingestion boundary.
- No browser workflow currently depends on canonical Contacts because the feature does not yet exist.

## Current FK/delete behavior reconfirmed

Current legacy behavior includes:

- `customers → whatsapp_connections`: CASCADE
- `conversations → customers/connections`: CASCADE
- `messages → customers`: SET NULL
- `messages → connections`: CASCADE
- outbound send structures restrict physical deletion of linked customer/connection records

The approved PR 1 architecture must therefore retain its canonical-history protection and must not make physical provider/legacy deletion silently erase Contact history.

## Live ingestion race

Live WhatsApp ingestion can create new customers after PR 1's migration-time backfill and before PR 2 adds Contact dual-write.

This remains an **intentional temporary gap**, not a blocker, because PR 1 exposes no Contacts UI or runtime dependency.

Operational requirements remain:

1. PR 1 is foundation only.
2. PR 2 follows promptly.
3. PR 2 adds atomic Contact/identity create-or-link behavior at the ingestion boundary.
4. PR 2 performs an idempotent catch-up backfill for customers created during the gap.
5. PR 2 must close/lock the ingestion boundary appropriately before its final catch-up validation so no customer escapes Contact coverage during the transition.
6. `/contacts` must not be enabled before PR 2 proves complete coverage.

## Final decision

### Blocking anomalies

**None.**

### Verdict

**GO**

### Authorization

**Contacts PR 1 — Foundation is authorized to begin using the approved `CONTACTS_TECHNICAL_SPEC.md` without architectural changes unless implementation reveals a new concrete blocker.**

Approved PR 1 boundaries remain:

- add `contacts`;
- add `contact_channel_identities`;
- additive/forward-only migration;
- backfill existing legacy customer(s);
- RLS + FORCE RLS + explicit privilege tests;
- browser read-only;
- existing `customers` retained;
- no `conversations` change;
- no `messages` change;
- no WhatsApp ingestion behavior change;
- no outbound-routing change;
- no AI behavior change;
- no `/contacts` UI yet.

## Next engineering step

Begin **Contacts PR 1 — Foundation** as a Draft PR, following the existing DBL large-PR workflow:

1. implementation;
2. focused/full validation;
3. self-review;
4. fix High/Medium findings;
5. independent final review;
6. squash merge only after approval;
7. production verification;
8. proceed promptly to PR 2 WhatsApp Contact Integration.
