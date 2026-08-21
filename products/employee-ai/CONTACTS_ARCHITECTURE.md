# Contacts Architecture — WhatsApp-first, Channel-ready

**Status:** Architectural direction approved for planning. Implementation has not started.

## Context

DBL Employee AI is currently focused on WhatsApp as the only supported customer messaging channel. The product vision still allows Instagram, Facebook Messenger and other channels later, but the current product must not spend engineering effort implementing integrations that are not needed yet.

The Contacts architecture therefore follows one rule:

> **WhatsApp is the only supported channel now, while the Contact domain must leave a clean path for future channels.**

This is deliberately **WhatsApp-first, channel-ready**, not multi-channel today.

## Why this decision exists

The current customer/messaging model was built around WhatsApp. Existing customer records are tied to WhatsApp-specific identifiers/connections, and conversations/messages currently operate through the WhatsApp path.

Building the Contacts UI directly around the assumption that `WhatsApp customer = permanent Contact model` would make future channel expansion unnecessarily expensive.

At the same time, building Instagram/Facebook ingestion, generic provider infrastructure, or speculative multi-channel abstractions now would over-engineer the MVP.

The chosen architecture keeps today's WhatsApp implementation focused while separating the long-lived concept of a Contact from channel-specific identity details.

---

## Architectural principles

### 1. Contact is a business/customer identity, not a WhatsApp account

The long-lived domain concept should be a Contact belonging to a Workspace.

Conceptually:

```text
Workspace
  └─ Contacts
       └─ Channel identity
            └─ WhatsApp  ← only supported channel today
```

A Contact must not permanently depend on fields such as WABA ID, WhatsApp connection ID, Instagram user ID, Facebook user ID, etc.

### 2. Channel-specific identity is separate

Channel-specific identifiers belong in an identity/link layer associated with the Contact.

Target conceptual shape:

```text
Contact
  ├─ id
  ├─ workspace_id
  ├─ display_name
  ├─ primary_phone (nullable)
  ├─ primary_email (nullable / future-use)
  ├─ first_seen_at
  ├─ last_seen_at
  ├─ created_at
  └─ updated_at

ContactChannelIdentity
  ├─ id
  ├─ workspace_id
  ├─ contact_id
  ├─ channel
  ├─ external_user_id
  ├─ channel_account_id / connection reference as appropriate
  ├─ username (nullable)
  ├─ display_name (nullable)
  ├─ phone_number (nullable)
  ├─ metadata (only when justified)
  ├─ first_seen_at
  ├─ last_seen_at
  ├─ created_at
  └─ updated_at
```

The exact schema is **not yet approved for migration**. These fields express the domain boundary, not a migration ready to run.

### 3. WhatsApp remains the only implemented channel

For the current product phase:

- WhatsApp Cloud API remains the only customer messaging integration.
- No Instagram ingestion is built.
- No Facebook Messenger ingestion is built.
- No TikTok/Threads/web/email integration is built.
- The Contacts UI should not advertise unsupported channels.
- WhatsApp-specific connection, Embedded Signup, WABA, webhook and credential infrastructure remains WhatsApp-specific.

We do **not** generalize `whatsapp_connections` merely for architectural symmetry. Provider connection abstractions should be designed only when another real channel is being implemented and its requirements are known.

### 4. Existing `customers` data is not deleted in the first migration

The existing WhatsApp customer model is production-sensitive and participates in ingestion/conversation behavior.

The first Contacts foundation must therefore be additive and backward-compatible.

Preferred transition concept:

```text
existing WhatsApp customer
        │
        ├── compatibility link
        ↓
canonical Contact
        │
        └── WhatsApp channel identity
```

The exact compatibility mechanism (`contact_id`, mapping table, or another reviewed approach) must be selected after a fresh implementation audit.

No destructive replacement of `customers` is allowed in the foundation phase.

### 5. Conversations should become Contact-aware gradually

Long-term desired relationship:

```text
Conversation
  ├─ contact_id
  └─ channel_identity_id
```

This answers separately:

- **Who is the customer?** → Contact
- **Through which channel identity did this conversation occur?** → Channel identity

The existing WhatsApp conversation path must continue to work during transition. Conversation/message rewiring must not be bundled casually into the first schema migration.

### 6. No unsafe automatic cross-channel merging

When future channels arrive, two identities must not be merged into one Contact merely because names look similar.

Example: a WhatsApp user named “Mohammed” and an Instagram user named “Mohammed” are not automatically the same person.

Future identity linking may use concepts such as:

- verified link
- high-confidence link
- suggested match
- unlinked

But automatic AI guessing is not part of the current architecture or MVP.

### 7. Workspace isolation remains authoritative

Contacts and channel identities are tenant data.

Any new tables must preserve DBL's existing security model:

- `workspace_id` scoped data
- RLS
- membership-derived authorization
- no trust in a browser-supplied workspace/contact identifier by itself
- no cross-workspace exposure of phone numbers, external identifiers, conversations, or customer history

### 8. Contacts UI must depend on a domain/query abstraction

The UI should not know whether Contact data currently comes from the legacy `customers` table, a future canonical `contacts` table, or a transitional join.

Introduce/maintain a server-side Contacts boundary returning product-oriented types such as:

```text
ContactSummary
ContactDetail
ContactConversation
```

Conceptually today:

```text
WhatsApp customer data → Contacts query/domain layer → Contacts UI
```

Later:

```text
contacts + channel identities → same Contacts query/domain layer → same Contacts UI
```

This protects the UX from future storage migrations.

---

## Refined invariants after architecture review

The following rules were added after an external critical review of the architecture. They are **pre-implementation invariants**, not final SQL decisions.

### Contact identity is always internal

- The canonical Contact identifier is an internal DBL UUID.
- Phone number, email, WhatsApp user ID, Instagram ID, username, or any provider identifier must never become the Contact primary key.
- Provider identifiers belong to the channel-identity layer and may change or multiply over time.

### A valid channel identity needs a trusted external identity

- `display_name` is descriptive data, not an identity key.
- Phone number is useful data, not a universal primary key.
- In the WhatsApp phase, a channel identity must have a provider-trusted external identifier sufficient to distinguish that customer within the relevant workspace/channel account scope.
- Missing display name or missing normalized phone must not by itself invalidate a legitimate provider identity.
- The UI must use safe fallbacks for missing names/phones rather than creating a new manual-review workflow merely for incomplete cosmetic data.

### Channel-identity creation/linking must be deterministic and idempotent

Webhook retries, concurrency, and repeated ingestion must not create duplicate channel identities or duplicate Contacts for the same trusted provider identity.

Conceptual invariant:

```text
workspace + channel + channel account/connection + external user identity
→ at most one active channel identity
```

The exact unique constraint/key shape must be decided by the PR 1 technical audit against the current production schema.

### Contact and ChannelIdentity are different domains even if today they are effectively 1:1

Current WhatsApp-only usage may produce one Contact per one WhatsApp identity, but application/domain code must not assume this remains permanently 1:1.

Future state may legitimately be:

```text
one Contact
  ├─ WhatsApp identity
  ├─ Instagram identity
  └─ Facebook identity
```

### Provider updates need precedence/provenance rules

Provider-derived attributes may change over time, for example WhatsApp display name or normalized phone information.

The architecture must support this principle:

- provider data can refresh provider-owned attributes;
- provider refreshes must not silently erase or overwrite future higher-confidence/manual business data;
- if DBL later allows business-edited Contact fields, the source/precedence of values must be explicit enough to prevent provider sync from undoing user intent.

A full provenance subsystem is **not required in Contacts MVP**, but PR 1/2 must avoid designs that make this impossible later.

### Disconnecting a channel does not delete the Contact or history

Disconnecting WhatsApp, removing a provider connection, or losing channel authorization is not equivalent to deleting the customer.

Therefore:

- Contact history must not be automatically deleted merely because a channel disconnects;
- the channel identity may become inactive/disconnected/historical according to the reviewed implementation;
- conversation/message history remains subject to the product's independent retention/deletion/privacy lifecycle;
- actual customer-data deletion must remain a separate explicit lifecycle decision.

### Legacy mapping is intentionally not pre-selected

The architecture does **not** pre-approve `legacy_customer_id`, `contact_id` on `customers`, or a mapping table.

PR 1 must first inspect the exact current schema, FKs, ingestion path, indexes, RLS, and rollback/backfill constraints and then recommend the safest compatibility mechanism.

Whatever mechanism is selected must be:

- additive initially;
- deterministic;
- idempotent;
- tenant-safe;
- compatible with existing production data;
- reversible/forward-recoverable without destructive customer-history rewriting.

### Conversation linking timing must be decided by the PR 1 audit

The long-term target remains:

```text
Conversation → Contact + ChannelIdentity
```

However, this document deliberately does not force `contact_id` or `channel_identity_id` into PR 2 before a schema audit.

PR 1 must recommend when and how conversations become Contact-aware, including nullable/backfill strategy if relevant, with explicit migration and rollback safety.

### Notifications are future scope, not a Contacts blocker

Creating a new Contact may later emit or feed a domain event that supports notifications, automations, Slack alerts, CRM integrations, etc.

Contacts MVP does not require notification infrastructure, and no event bus should be introduced solely for speculative future use.

---

## Implementation roadmap

### Contacts PR 1 — Foundation

Goal: establish the canonical Contact/channel-identity foundation safely.

Expected work after a fresh schema audit:

- additive schema only
- canonical Contact model
- WhatsApp channel identity model
- workspace-scoped indexes and RLS
- safe backfill plan for existing WhatsApp customers
- compatibility-link decision based on the audited production schema
- deterministic/idempotent migration behavior
- explicit uniqueness semantics for trusted WhatsApp identities
- missing-data/fallback behavior
- provider-update precedence strategy sufficient for MVP safety
- explicit recommendation for when/how conversations become Contact-aware
- database tests for tenant isolation, concurrency/idempotency and backfill correctness

No Contacts UI is required in this PR.

### Contacts PR 2 — WhatsApp integration

Goal: make current WhatsApp ingestion create/link the new Contact model without changing customer-visible WhatsApp behavior.

Requirements:

- existing inbound ingestion remains reliable
- existing customers map safely to Contacts
- new WhatsApp customers create/link Contacts and WhatsApp identities
- retries remain idempotent
- concurrent/repeated ingestion cannot silently create duplicate identities/Contacts
- missing name/phone data fails gracefully when provider identity is valid
- provider-derived attribute refreshes respect the approved precedence rules
- channel disconnect does not erase Contact/history
- conversations remain correctly associated according to the migration strategy approved from PR 1
- no provider configuration changes unless explicitly required and reviewed

### Contacts PR 3 — Contacts MVP UI

Goal: expose the customer memory view using the new domain boundary.

Routes:

```text
/contacts
/contacts/[contactId]
```

The MVP is read-only.

List page direction:

- title + short explanation
- total Contacts
- new Contacts over a clearly defined period
- returning Contacts only if the metric can be defined truthfully from current data
- search by name or phone
- server-side pagination
- desktop table
- mobile cards at 360px
- no horizontal overflow

Contact detail direction:

- profile header
- WhatsApp identity/phone
- first interaction
- last interaction
- conversation count
- conversation history/timeline
- link to existing conversation details
- customer information snapshot

### Contacts PR 4 — Legacy reduction, only after proven stability

After the new Contact path is proven in production, independently review whether the old `customers` dependency should be:

- removed,
- reduced,
- retained as a WhatsApp-specific compatibility/ingestion entity,
- or migrated further.

There is no pre-approved requirement to delete `customers`.

---

## Contacts MVP UX boundaries

The first Contacts release is intentionally not a full CRM.

### Included

- automatically discovered Contacts from WhatsApp interactions
- name / safe fallback identity
- WhatsApp phone/identifier
- first seen
- last seen
- conversation count
- latest conversation/status where reliable
- search
- pagination
- linked conversation history
- Arabic + English
- RTL/LTR correctness
- accessible desktop and 360px layouts
- owner/admin/agent/viewer read access according to existing Workspace membership protections

### Explicitly deferred

- manual Add Contact
- Edit Contact
- Delete Contact
- Merge Contact
- tags
- internal notes
- lead stages
- custom fields
- bulk actions
- CSV import/export
- AI customer summary
- contact scoring
- manual cross-channel linking
- Instagram/Facebook UI
- team notifications on Contact creation

These may be designed later after real customer usage validates the need.

---

## UX philosophy

Contacts is not a phone book.

It should answer three questions quickly:

1. Who is this customer?
2. When/how have they interacted with the business?
3. What conversation history exists with them?

Product distinction:

- **Conversations** = what is happening in customer communication.
- **Contacts** = who the customer is and the history of the relationship.

The Contacts page should feel like a lightweight customer-memory surface, not an enterprise CRM dashboard.

### `/contacts` design direction

Desktop:

- page header and explanation
- compact useful metrics
- search
- modern table
- whole row opens Contact detail
- server-side pagination, default target around 25 rows/page unless implementation measurements justify another value

Mobile:

- cards instead of a compressed table
- no horizontal navigation/data overflow at 360px

Empty state when WhatsApp is connected:

> Contacts will appear here automatically when customers begin messaging the business on WhatsApp.

When WhatsApp is not connected, guide the owner/admin toward WhatsApp setup rather than offering manual Contact creation.

### `/contacts/[contactId]` design direction

- avatar/fallback initials
- name
- WhatsApp phone/identity
- first interaction
- last interaction
- conversation count
- action to open the latest conversation where available
- conversation history ordered by recent activity
- customer information snapshot

The layout may reserve structural room for future tags/notes/customer memory, but **must not render fake placeholders or Coming Soon CRM features in the MVP**.

---

## Internationalization and accessibility

Required:

- Arabic RTL
- English LTR
- phone numbers rendered LTR even inside Arabic UI
- customer names use direction-aware rendering (`auto` where appropriate)
- locale-aware dates
- keyboard-accessible Contact rows/cards/actions
- visible focus states
- status must not rely on color alone
- semantic controls rather than clickable non-semantic containers
- safe user-facing errors with no SQL/Supabase/internal terminology

---

## Performance rules

Do not preemptively add a separate search engine or embeddings for Contacts.

Start with bounded server-side queries and pagination.

Only add specialized indexes (for example trigram search) after measuring real query behavior and approving a focused migration.

Avoid expensive per-contact message counting in the list unless query plans/data volumes prove it safe.

Analytics-style time-series aggregation belongs to Analytics, not Contacts.

---

## Relationship to Analytics

Contacts should be established before Analytics metrics that depend on customer identity.

A canonical Contact concept makes future metrics such as these meaningful:

- unique Contacts
- new Contacts
- returning Contacts
- conversations per Contact
- Contacts by channel when additional channels actually exist
- multi-channel Contacts in the future

Analytics must not treat multiple channel identities as multiple people once cross-channel identity support exists.

---

## Future Customer Memory

A future version may allow the AI Employee to use customer-specific memory, such as verified preferences or prior purchases, under explicit privacy/security controls.

This is **not part of Contacts MVP**.

If implemented later, avoid an opaque free-form `ai_memory` blob as the sole source of truth. Prefer structured facts/events with provenance and lifecycle fields so the system can explain where a fact came from and when it should expire or be reviewed.

Conceptually future-only:

```text
fact
source/provenance
confidence or verification state
created_at
expires_at / lifecycle
```

This should follow the same philosophy as DBL Knowledge grounding: traceable information before unsupported claims.

---

## Separate future idea: Team Workspace

A related product idea has been captured but is **not part of the Contacts implementation**:

- a business owner owns the Workspace
- owner invites multiple people using separate user accounts
- owner controls roles/permissions
- potential paid plans/seats for team access
- future conversation/contact assignment to human team members

The existing Workspace membership/role foundation should be reused rather than creating shared passwords or a single shared login.

Do not let this idea expand Contacts scope now.

---

## Current stopping point

As of **2026-08-11**:

- The Contacts architectural direction is agreed.
- Product remains WhatsApp-only for customer messaging today.
- Future channel expansion is intentionally allowed by the Contact domain design.
- Existing WhatsApp-specific infrastructure should remain provider-specific for now.
- The Contacts MVP UX direction has been defined.
- Critical-review refinements have been added: UUID identity, missing-data rules, uniqueness/idempotency, provider-data precedence, disconnect retention, legacy-mapping audit requirement, and conversation-linking decision gate.
- No Contacts implementation has started from this plan yet.
- Next engineering step: perform a fresh implementation/schema audit against current `main`, convert this architecture into an exact Technical Specification, then implement in small independently reviewed PRs beginning with the Contact Foundation.

### Decision summary

> **Build Contacts as a WhatsApp-first customer-memory feature with a channel-ready domain boundary. Do not implement future channels now, do not prematurely generalize WhatsApp provider infrastructure, and do not lock the long-lived Contact identity to WhatsApp-specific storage. Contact identity is internal, channel identity is provider-scoped, creation/linking must be idempotent, and legacy/conversation migration details must be approved only after a fresh technical audit.**
