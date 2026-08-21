# Contacts PR 1 Technical Specification

**Status:** Approved for preflight. Implementation must not begin until the production read-only preflight passes.

**Date:** 2026-08-11

## Purpose

This document converts the approved `CONTACTS_ARCHITECTURE.md` direction into an implementation-ready technical specification for **Contacts PR 1 — Foundation**.

It is based on a read-only audit of current `dbl-employee-ai` `main` at:

`d4d5dbfa988dc2e653f0164193fb9df5a4aba5f2`

The audit was performed by Codex and independently reviewed by DBL planning plus DeepSeek. No implementation changes were made during the audit.

---

## Approved scope

PR 1 is foundation only.

It should:

- add canonical `contacts`;
- add `contact_channel_identities`;
- use internal UUID Contact IDs;
- represent only WhatsApp as a supported channel today;
- preserve existing `customers` unchanged;
- backfill one Contact + WhatsApp identity for each existing WhatsApp customer;
- provide workspace-scoped RLS/read access;
- deny normal browser writes;
- leave WhatsApp ingestion behavior unchanged;
- leave `conversations` unchanged;
- leave `messages` unchanged;
- leave outbound routing unchanged;
- leave AI behavior unchanged;
- expose no Contacts UI yet.

PR 1 must be additive and forward-only.

---

## Current-system findings driving the design

The existing `customers` table is not display-only. It participates in:

- inbound WhatsApp customer upsert;
- conversation creation/reuse;
- conversation queries;
- dashboard metrics;
- trusted outbound destination routing;
- outbound send reservations;
- existing SQL and E2E fixtures.

Therefore it must not be destructively replaced in PR 1.

The current WhatsApp connection UUID may be reused while provider account metadata changes. For identity uniqueness, the audit therefore recommends snapshotting the receiving Meta `phone_number_id` as the WhatsApp account scope rather than treating `whatsapp_connection_id` as the immutable provider-account identity.

---

## Canonical Contact model

Approved minimal shape:

```text
contacts
- id uuid primary key
- workspace_id uuid not null
- display_name text null
- first_seen_at timestamptz not null
- last_seen_at timestamptz not null
- created_at timestamptz not null
- updated_at timestamptz not null
```

Approved rules:

- `id` is the canonical Contact identifier.
- Phone number is not a Contact primary key.
- Email is not a Contact primary key.
- WhatsApp user ID is not a Contact primary key.
- Provider IDs do not define the Contact itself.
- `primary_phone` is intentionally omitted from PR 1.
- CRM fields such as tags, notes, lifecycle stage, owner, address, consent, custom fields, company and scoring are deferred.
- `display_name` maximum: 160 characters when present.
- `last_seen_at >= first_seen_at`.
- Workspace-scoped uniqueness/supporting key may be added as needed for tenant-consistent composite FKs.

---

## Channel identity model

Approved conceptual shape for PR 1:

```text
contact_channel_identities
- id uuid primary key
- workspace_id uuid not null
- contact_id uuid not null
- channel text not null
- channel_account_external_id text not null
- external_user_id text not null
- whatsapp_connection_id uuid not null
- provider_display_name text null
- phone_number_normalized text null
- legacy_customer_id uuid null
- first_seen_at timestamptz not null
- last_seen_at timestamptz not null
- created_at timestamptz not null
- updated_at timestamptz not null
```

For PR 1:

- `channel = 'whatsapp'` only.
- `channel_account_external_id = whatsapp_connections.phone_number_id`.
- `external_user_id = customers.whatsapp_user_id`.
- `legacy_customer_id = customers.id`.

No Instagram, Facebook Messenger, TikTok, Threads, web-chat or email provider implementation is added.

---

## Identity uniqueness invariant

Approved logical invariant:

```text
workspace_id
+ channel
+ channel_account_external_id
+ external_user_id
= one channel identity
```

Purpose:

- prevent duplicate identities under retries/concurrency;
- separate the same external user across different receiving WhatsApp account scopes;
- preserve future channel extensibility;
- avoid relying on display names or phone matching.

Also require one legacy WhatsApp customer to map to at most one compatibility identity.

No name-based automatic merge is permitted.

---

## Legacy compatibility decision

Options reviewed:

1. `customers.contact_id`
2. `contacts.legacy_customer_id`
3. separate mapping table
4. `contact_channel_identities.legacy_customer_id`

**Approved choice:** `contact_channel_identities.legacy_customer_id`.

Why:

- legacy `customers` stays untouched;
- the legacy WhatsApp customer maps to its exact WhatsApp identity;
- the identity owns the Contact relationship;
- no second competing Contact link is introduced on `customers`;
- no third mapping table is required;
- future legacy removal is clearer.

Expected transitional lookup:

```text
conversation.customer_id
→ contact_channel_identities.legacy_customer_id
→ contact_channel_identities.contact_id
→ contacts.id
```

---

## Backfill design

Backfill must be deterministic, additive, and non-merging.

For every existing customer at migration time:

1. create one Contact;
2. create one WhatsApp channel identity linked to that Contact;
3. copy `profile_name` into Contact `display_name` and identity `provider_display_name` when present;
4. copy `phone_number_normalized` when present;
5. copy first/last-seen timestamps;
6. snapshot current receiving `phone_number_id` into `channel_account_external_id`;
7. set `legacy_customer_id` to the existing customer UUID.

Rules:

- missing `profile_name` is allowed;
- missing normalized phone is allowed;
- valid WhatsApp external identity remains sufficient;
- duplicate-looking names stay separate;
- multiple WhatsApp connections/account scopes stay separate;
- disconnected connections/history are still backfilled;
- no automatic cross-connection merge;
- no name-based matching;
- collisions/anomalies abort instead of choosing a winner.

A genuine migration-upgrade test must seed legacy rows before applying the Contacts migration. A clean database reset alone is insufficient to prove production backfill behavior.

---

## Provider-derived data precedence

PR 1 does not change ingestion behavior.

Approved rule for PR 2 and later:

- provider-derived values update Channel Identity data;
- an empty provider name never erases an existing non-empty provider name;
- Contact `display_name` may follow provider refreshes only while it is null or still equals the previous provider-derived value;
- once future manual editing changes Contact `display_name`, provider refresh must not overwrite it;
- Contact `first_seen_at` tracks the earliest known identity activity;
- Contact `last_seen_at` tracks the latest known identity activity;
- provider phone changes belong to Channel Identity, not a manually controlled Contact-level phone field.

A complex provenance table is not required in PR 1.

---

## Disconnect and history rules

Approved:

- WhatsApp disconnect does not delete Contact;
- disconnect does not delete Channel Identity;
- disconnect does not delete legacy customer;
- disconnect does not delete conversations or messages;
- history remains visible/retained;
- disconnected identity is not usable for sending through an inactive connection;
- reconnection to the same provider account may reuse the identity where technically valid;
- a different receiving `phone_number_id` is a different account scope;
- Contact deletion is unsupported in MVP.

Physical deletion of connection/customer structures must never silently cascade away canonical Contact history.

---

## Conversation and message decision

### PR 1

**Conversations:** unchanged.

Do not add `contact_id` or `channel_identity_id` in PR 1.

Reason:

- adding them creates dual-write/consistency responsibility immediately;
- current inbound/outbound code is deeply coupled to `customer_id`;
- PR 1 can resolve Contact through the compatibility identity join;
- broad rewiring would raise migration and rollout risk materially.

### Messages

No Contact-related schema change in PR 1.

Message-level Contact duplication provides no concrete benefit now and creates consistency risk.

### Later

After Contact/identity writes are proven, a separate reviewed migration may add:

```text
conversation.contact_id
conversation.channel_identity_id
```

if the implementation audit at that time confirms this remains the best design.

---

## RLS and permissions

Approved for both new tables:

- workspace-scoped tenant data;
- RLS enabled;
- FORCE RLS;
- no cross-workspace reads;
- no browser-trusted workspace identifier as authorization proof;
- active workspace membership controls reads;
- Owner/Admin/Agent/Viewer may read Contacts within their active Workspace;
- suspended/non-members cannot read;
- no authenticated browser INSERT/UPDATE/DELETE in PR 1;
- provider/ingestion writes remain server/service-controlled;
- phone numbers and provider identifiers remain tenant-private;
- no new browser-executable SECURITY DEFINER RPC needed in PR 1.

Privileges and RLS must both be tested explicitly.

---

## Index strategy

Only justified foundation indexes should be added.

Expected:

### contacts

```text
PRIMARY KEY (id)
UNIQUE (workspace_id, id)
INDEX (workspace_id, last_seen_at DESC, id DESC)
```

### contact_channel_identities

```text
PRIMARY KEY (id)
UNIQUE (workspace_id, channel, channel_account_external_id, external_user_id)
UNIQUE (legacy_customer_id) WHERE legacy_customer_id IS NOT NULL
INDEX (contact_id)
```

Potential tenant-pair unique indexes may be added to existing `customers` / `whatsapp_connections` only where required for composite FKs.

Do not add in PR 1:

- trigram indexes;
- full-text search;
- search engine infrastructure;
- generalized JSON indexes;
- analytics indexes;
- speculative phone/email search indexes.

---

## Exact PR 1 migration sequence

Approved intended order, subject to preflight findings:

1. begin explicit transaction where operationally safe;
2. validate existing customer/connection invariants;
3. add required tenant-pair unique indexes to legacy tables if needed;
4. create `contacts`;
5. add Contact constraints and updated-at behavior;
6. create `contact_channel_identities`;
7. add tenant-consistent FKs, channel/timestamp/length checks and uniqueness constraints;
8. enable + force RLS;
9. revoke public/browser write privileges;
10. add authenticated read policies/grants;
11. add minimal service-role write privileges;
12. construct deterministic migration-local legacy mapping;
13. insert Contacts;
14. insert channel identities;
15. validate exact coverage, no cross-workspace links, no orphans and matching counts;
16. add only justified indexes not already required earlier;
17. commit;
18. regenerate/review database types;
19. add migration-upgrade + pgTAP/RLS tests.

No existing migration file is edited.

---

## Preflight is a mandatory Go/No-Go gate

Implementation must not begin until a **read-only production preflight** confirms:

- production customer row count and expected migration size;
- zero customer/workspace vs connection/workspace mismatches;
- zero customers attached to a connection lacking `phone_number_id`;
- zero normalized proposed identity collisions;
- no legacy anomaly requiring product-specific remediation.

If any anomaly is found:

> **STOP. Do not auto-fix, auto-merge or silently normalize records.**

A remediation plan must be reviewed separately.

The preflight may run directly against Production because it is read-only. No production writes or schema/config changes are authorized by the preflight.

---

## PR 1 → PR 2 gap

Known intentional gap:

Customers created after the PR 1 migration snapshot and before PR 2 dual-write integration may temporarily have no canonical Contact/identity.

This is acceptable only because:

- PR 1 exposes no Contacts UI;
- no existing runtime behavior depends on canonical Contacts yet.

Operational rule:

- PR 2 follows PR 1 directly / within a short engineering window;
- PR 2 must include catch-up backfill for customers created during the gap;
- `/contacts` must not be enabled until PR 2 dual-write + catch-up coverage is proven.

Do not interleave unrelated broad feature work between PR 1 and PR 2 if avoidable.

---

## Performance rule

Backfill strategy depends on real production volume.

If the production customer count is small/moderate, keep the migration simple.

If volume or anomaly scans show meaningful lock/WAL/table-scan risk, redesign rollout before implementation using an online/batched strategy.

Do not pre-build complex batching/monitoring infrastructure without evidence that it is needed.

---

## Required PR 1 tests

### Migration-upgrade

Cover:

- normal legacy customer;
- missing profile name;
- missing normalized phone;
- duplicate-looking names;
- multiple WhatsApp connections/account scopes;
- disconnected connection;
- same external ID in different Workspaces;
- same external ID under different account scopes;
- rerun-safe/idempotent behavior where applicable;
- legacy customers/conversations/messages unchanged.

### RLS / pgTAP

Cover:

- Owner read;
- Admin read;
- Agent read;
- Viewer read;
- suspended member denied;
- outside user denied;
- cross-workspace read denied;
- all normal authenticated browser writes denied;
- anon denied;
- server/service path write allowed only as intended;
- cross-workspace FKs rejected;
- duplicate identity rejected;
- duplicate `legacy_customer_id` rejected;
- disconnect preserves Contact/identity;
- deletion cannot silently remove linked canonical history.

### Regression

Existing suites must prove unchanged behavior for:

- inbound WhatsApp ingestion;
- message idempotency;
- outbound trusted routing;
- AI generation/automatic reply behavior;
- tenant isolation;
- Embedded Signup;
- authenticated E2E.

---

## Explicit PR 1 boundaries

Do not change in PR 1:

- WhatsApp parser/ingestion runtime behavior;
- outbound messaging runtime behavior;
- webhook route;
- conversation UI/actions;
- dashboard UI/metrics;
- AI generation/retrieval/grounding;
- WhatsApp Settings / Embedded Signup;
- app-shell/navigation;
- auth/legal;
- Knowledge workflows;
- Meta/Vercel/Google WIF configuration;
- Contacts UI;
- existing historical migration files.

---

## Approved product decisions

1. All active Workspace roles, including Viewer, may read Contacts in the same Workspace for MVP.
2. `phone_number_id` is approved as the current WhatsApp account-scope snapshot used in identity uniqueness, based on the audited current reconnection model.
3. No automatic Contact merge in MVP.
4. No manual Contact merge in MVP.
5. Contact deletion is unsupported in MVP.
6. Disconnect preserves Contact, identity, conversations and history.
7. Provider display-name refresh may update Contact name only while that Contact name remains provider-controlled (null or matching previous provider value).
8. Production preflight is mandatory before migration implementation.
9. PR 2 must directly close the PR 1 dual-write gap before Contacts UI is enabled.
10. Migration complexity/performance strategy is chosen from actual production preflight volume, not speculation.

---

## Independent review outcome

### Codex audit

Result: Technical design recommended with PR 1 risk **4/10 (low-to-medium)**, conditional on production preflight.

### DeepSeek independent review

Result: Architecture and technical audit judged sound and implementation-ready, with emphasis on:

- formally answering open product questions;
- treating preflight as mandatory;
- keeping PR 2 close behind PR 1;
- monitoring migration/backfill risk proportionally to actual data volume.

DBL accepted these recommendations with the refinement that monitoring/batching complexity should only be introduced if production volume warrants it.

---

## Current state

**Architecture:** Approved.

**Technical Specification:** Approved.

**Implementation:** Not yet authorized.

**Next step:** Read-only Production Contacts Preflight.

If preflight returns zero blocking anomalies, authorize Contacts PR 1 Foundation implementation.
