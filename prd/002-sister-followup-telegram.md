# PRD-002: Sister Follow-Up over Telegram (MVP)

| | |
| --- | --- |
| **Status** | Draft |
| **Date** | 2026-09-26 |
| **Product owner** | Didi Ananda Devapriya (Acharya) |
| **Build owner** | Taras Kornichuk |
| **Operator** | Madhu (assistant), or the next assistant |
| **Refines** | [PRD-001](001-initiate-support-crm.md). Section 6 lists the deltas. |

## Sources

| Source | What it gave |
| --- | --- |
| Working session, 2026-09-26, 35 min (Svidok #1306; transcript and digest in Allspark `90-sources/…/2026-09-26-initiate-crm-telegram-followups.md`) | Decisions, scope, pain points |
| Airtable base "Didi's Initiation Database" (`appaCa0vfJRfpU4pF`), live schema and aggregate counts read 2026-09-26 | Real data model, automations, volumes |
| `Acarya Airtable CRM Process Documentation.docx` | Documented SOPs for the Airtable base |
| PingCRM code at `main@7537ca9` | What we can reuse |
| [PRD-001](001-initiate-support-crm.md) | First vision draft |

---

## 1. Problem

The Airtable base works for intake, lesson pipelines, RDS reports and donations. Didi wants to keep it. It fails at one job: **follow-up**. Sisters "fall through the cracks".

| Pain | Evidence |
| --- | --- |
| Templates exist, but only for email, one sister at a time | 13 rows in `Message Templates`. The "Send Email" button opens a mail client for one record. The bodies have no placeholders. |
| No path to Telegram | Didi copies template text into Telegram by hand. 410 of 608 sisters have a Telegram ID. Telegram is the main channel for Ukraine, Moldova and most of the Russian-speaking group. |
| Retreat reminders are off | The 9 "Retreat – Students" automations (T-60, T-30, T-7 per country) are OFF. Airtable caps a send at 100 records, and the plan limits email volume. Germany alone has 161 sisters. |
| Status goes stale | "Change status to unknown after 3 months" runs on every record. Today: 201 `Unknown`, 127 empty, 117 `Active`. |
| No shared assistant channel | Madhu uses a personal account. If Madhu stops, the history goes too. |
| New sisters do not reach the phone | Someone must add each new sister to Didi's (or Madhu's) address book by hand. Didi stopped this in summer 2026. |
| Tour reminders are manual | When Didi plans a trip, nobody prompts the assistant to contact the sisters in that country. |

## 2. Goals and non-goals

### Goals (MVP)

| ID | Goal | Target |
| --- | --- | --- |
| G1 | The assistant selects a segment, selects a template, previews each message, and sends through the assistant Telegram account | 50 sisters in less than 5 min of human time |
| G2 | A tour or retreat in Didi's calendar creates reminder batches for the sisters in that place | Batches ready at T-30, T-14, T-7 with no manual set-up |
| G3 | Each new sister appears in the assistant's Telegram contacts and in a "Sisters" Google Contacts label | Less than 1 h after the Airtable record appears |
| G4 | The system holds sister data only | Zero non-sister dialogs stored |
| G5 | A new assistant can take over | The account, the history and the templates belong to the workspace, not to a person |

### Non-goals (MVP)

- Rebuild the Airtable intake forms, lesson pipelines, RDS dashboard or donations. They stay in Airtable. [PRD-003](003-forms-module-and-module-system.md) covers the later move of forms.
- Prachar CRM, event leads, cold leads, brothers who did not take a mantra.
- Sync of Didi's personal Telegram history.
- Automatic sends with no human review.
- WhatsApp (needed for Romania later).
- Several Acharyas in one install. We design for it (section 8.4) but do not build it.
- The voice-message bot that lets Didi request changes (separate TAO project).

## 3. Users and roles

| Role | Person | Needs | MVP access |
| --- | --- | --- | --- |
| `owner` | Didi (Acharya) | Sees who needs contact; owns templates; does not want to operate the tool daily | All data, all settings |
| `assistant` | Madhu | Daily sends, batches, replies, contact provisioning | Sister profiles and messages. No `Acarya Notes`, `Asanas Notes`, `Ask Didi`. |
| `admin` | Taras | Set-up, integrations, fixes | Settings and logs. Sister data only when necessary. |
| `country_leader` | Six leaders in `Country leaders` | Quarterly status check (Airtable does this now) | Not in MVP |
| Sister | Initiate | Feels remembered; gets timely invitations in her language | Receives messages only |

## 4. Current state

### 4.1 Airtable base (stays the system of record for initiation data)

Aggregate counts, read 2026-09-26. No personal data copied.

```
Sisters ................ 608 rows, 67 fields, 21 views   (created 2026-01-24 .. 2026-09-26)
Higher Lessons Requests   61     Retreats ............ 11
Message Templates ....... 13     Country leaders ...... 6
Donations (initiates) ...  7     Country Leaders update 2
Donations (others) ......  0
Interfaces: Nama Mantra Pipeline, 1st Lesson Pipeline, Lesson review requests,
            Higher Lessons Pipeline, Contacts, Insights & Reporting (RDS), Donations x2,
            AI-generated elements (9 pages)
Automations: 37 total, 28 ON, 9 OFF (all "Retreat – Students")
Collaborators: 1 owner, 5 read, 4 none
```

| Dimension | Distribution |
| --- | --- |
| Country | Germany 161, Ukraine 157, Romania 43, Czech 32, Spain 22, Moldova 18, Brazil 17, Netherlands 17, Bulgaria 14, Taiwan 11, 20 others ≤ 9, empty 59 |
| Lesson Received | Nama 250, 1st 135, 2nd 77, 3rd 38, 4th 16, 5th 14, 6th 30, empty 48 |
| Preferred Language | Russian 232, Ukrainian 170, Romanian 44, English 13, Dutch 12, Taiwanese 10, German 8, other 17, empty 102 |
| Contact fields filled | Email 510, Phone 441, Telegram ID 410, Birthday 313, Date of Nama 271, Date of Initiation 248 |
| Do Not Contact | 1 |

**Fields the MVP reads from `Sisters`:** `First Name` / `Last Name` / `City` (formulas that prefer the Romanised value), `Sanskrit Name`, `Email Address`, `Clean Email`, `Phone Number`, `@Telegram ID`, `Country`, `Preferred Language`, `Birthday`, `Date of Nama`, `Date of Initiation`, `Lesson Received`, `Date of Lesson Received`, `Lessons Reviewed`, `Lesson Requested`, `Date of Lesson Request`, `Init - Margii Stage`, `Nama to Init stage`, `Status`, `Marital Status`, `Source`, `Project`, `Duty`, `Do Not Contact`.

**Fields the MVP does not read:** `Acarya Notes`, `Asanas Notes`, `Ask Didi`, `Profile Photo`, donation links, `Expertise / Interests`, `Birthplace`.

**Automations that already work (keep):** 3 × Cyrillic converter per form type (Russian, Ukrainian), birthday / Nama / Diksha greeting emails and Monday digests, quarterly country-leader reminders (5 countries), donation acknowledgements, "Nama to Init stage" and "Date of Init" auto-updates, 3 × retreat reminders to the assistant.

**Data-quality problems the sync must handle:**

- Duplicate or misspelt language options: `Russian` / `Russian ` / `Russish`, `Romanian` / `Romana`, `Italian `, `Dutch `, `Lativian`, `Lthuania - Russian`, `france`.
- `Ukraine` option has trailing spaces. `Greece ` too.
- Email is the only join key between `Sisters` and `Higher Lessons Requests`. A request with a new or wrong email matches no sister and does not show in any interface.
- The Cyrillic converter does not run on some records (reported in the session).
- `@Telegram ID` is free text: it can hold `@username`, a phone number, or a name.

### 4.2 PingCRM today

PingCRM is a fork of `sneg55/pingcrm` (AGPL-3.0), a single-user networking CRM. Paths are relative to `backend/app/` unless noted.

| Area | What exists | Gap for this PRD |
| --- | --- | --- |
| Tenancy | Every row has `user_id`; every query and MCP tool filters on `current_user.id` (`api/interactions.py:68-70`). `ALLOW_REGISTRATION=False`; the config calls the product "single-player" (`core/config.py:26-29`). | No workspace, members, roles or sharing |
| Contact model | `contacts` has names, `emails[]`, `phones[]`, `telegram_username`, `telegram_user_id`, `location` + geocode, `birthday` (string), flat `tags[]`, one `notes` field, `priority_level`, `relationship_score` (`models/contact.py:17-102`) | No custom fields, language, lesson, stage, country field, or external ID |
| Telegram session | One Telethon session per user, encrypted on `User.telegram_session`. Login: phone → code → 2FA (`integrations/telegram.py:115-210`). | Session is on a user, not on a workspace |
| Telegram sync | Polls once a day at 03:00 UTC; 1:1 chats, last 50 messages, text cut at 500 chars, media dropped. **Creates a contact for every 1:1 chat** (`integrations/telegram_chat_sync.py:118-139`). Group sync adds members as "2nd Tier" contacts (`integrations/telegram_groups.py:75`). | Violates D4 unless we switch to "match only, never create" |
| Telegram send | `POST /contacts/{id}/send-message` sends text and logs an outbound interaction (`api/contacts_routes/messaging.py:38-143`). FloodWait gate in Redis (`integrations/telegram_transport.py:17-37`). | **Needs `telegram_username`** (`messaging.py:69-74`); many sisters have only a phone. No batch, no queue, no daily cap. |
| Interactions | `platform`, `direction`, `content_preview`, `occurred_at`, `extra_data` | No sender or account, no template ID, no delivery state |
| Suggestions | Daily engine, cadences 30/60/180 days by priority, snooze, birthday trigger; needs an earlier interaction and an English prompt (`services/followup_engine.py:76-91,193`; `services/message_composer.py:364-400`) | No stage or date-offset rules; English only |
| Filters | Search, **one** tag, priority, score, dates, birthday (`api/contacts_routes/listing.py:28-75`); bulk tag/priority up to 500 | No saved segments, no multi-field filters, no bulk send |
| Import | Fixed-column CSV, LinkedIn CSV, Google Contacts read | No Airtable import, no CSV export |
| Google | Scopes `contacts.readonly`, `calendar.readonly`, `gmail.*` (`integrations/google_auth.py:11-18`). Calendar sync runs at 06:00 and **turns event attendees into contacts** (`integrations/google_calendar.py:161`). | Contacts write needs the `contacts` scope. Calendar sync must read events, not create contacts. |
| Notifications | In-app table and page. Weekly digest is logged, **not sent** (`services/digest_email.py:128,191`). | No push to the assistant |
| WhatsApp | whatsapp-web.js sidecar, real-time receive, **no send** | Out of scope (Q7) |
| i18n | None. `<html lang="en">`, hard-coded English | Q5 |
| Tests | Backend 102 files (~1,318 tests, real Postgres); frontend 37 Vitest files | Good base for TDD |

**Reuse:** Telethon login, encrypted sessions, the send function, the FloodWait gate, the interaction timeline, Celery + beat, the Claude drafting code, the Google OAuth flow, the map.

**Switch off for this workspace:** auto-create of contacts from chats, groups and calendar attendees; Twitter, LinkedIn, Meta, Apollo, Gmail sync; the Chrome extension. They add risk and do not serve sisters.

## 5. Decisions from the 2026-09-26 session

| # | Decision | Effect on this PRD |
| --- | --- | --- |
| D1 | Keep the Airtable flow. Do not rebuild it 1:1. | Airtable is the source; PingCRM reads it. |
| D2 | Build new functions first: follow-up automation and integrations. | Scope = sections 7.1–7.6 |
| D3 | Do not sync Didi's whole Telegram history. | Only the new assistant account connects. |
| D4 | Scope = sisters only. No Acharya, family or personal contacts. | Sync filter (7.2) |
| D5 | No second SIM for Didi. | Didi keeps one identity. |
| D6 | Buy a fresh SIM, create a clean Telegram account for the assistant. | Account belongs to the workspace (G5). |
| D7 | Prachar CRM stays out of scope. | Non-goal |
| D8 | Didi's data must not depend on an external system (for example the Dharmacakra bot CRM). | Self-hosted; integrations are optional. |
| D9 | Each Acharya needs a private, locked space ("an apartment in a building"). | Workspace + RBAC (8.4) |

## 6. Deltas from PRD-001

| PRD-001 item | Change | Why |
| --- | --- | --- |
| Epic 1: connect Didi's personal Telegram as Account 2 | **Removed** from MVP | D3, D5 |
| Epic 1.3: escalation queue with voice reply from the CRM | **Reduced** to a "Needs Didi" flag + note; Didi replies from Didi's own phone | Smaller first step |
| Epic 2: `InitiateProfile` with `lesson_1…6`, `sadhana_status` | **Replaced** by the real Airtable fields (section 4.1) | The fields already exist and have data |
| Epic 3: content tiers | **Kept** as the audience rule on each template (7.4) | Cheap guard, avoids wrong-tier content |
| Epic 4.1: 7-day challenge webhook | **Deferred** | No 7-day bot in use today |
| Epic 4.2: Lesson-1 cadence (Day +3, +30, +60) | **Kept** as a P1 rule set (7.7) | Matches "invite to Dharmacakra / 2nd lesson" templates |
| Epic 4.3: tour outreach | **Kept**, now driven by Google Calendar (7.6) | Session request |
| Epic 5.1: one-time Airtable import | **Replaced** by a continuous one-way sync (7.1) | D1: Airtable stays live |
| Epic 5.2: SOP for other Acharyas | **Moved** to [PRD-003](003-forms-module-and-module-system.md) (open-source packaging) | |

## 7. Functional requirements

Priority: **P0** = MVP must have. **P1** = MVP should have. **P2** = after MVP.

### 7.1 Airtable sync (P0)

- One-way pull from `Sisters` into PingCRM contacts every 15 min, plus a "Sync now" button.
- Auth: Airtable personal access token, scope `data.records:read` + `schema.bases:read`, stored encrypted.
- Stable key: the Airtable record ID (`rec…`). Store `Clean Email` as a second key only.
- Map by Airtable **field ID**, not by name. A rename must not break the sync. A missing field raises a visible sync error.
- Normalise `Preferred Language` to ISO codes (`ru`, `uk`, `ro`, `en`, `de`, `nl`, `zh-TW`, …) with a fixed map for the misspelt options.
- Trim all select values.
- Parse `@Telegram ID` into `telegram_username` or `telegram_phone`. Keep the raw value.
- A record deleted in Airtable becomes `archived` in PingCRM. Never hard-delete.
- `Do Not Contact = true` excludes the sister from every send path.
- Airtable-owned fields are read-only in PingCRM. The UI shows an "Edit in Airtable" link.
- **Acceptance:** 608 records sync in less than 60 s. A second run with no changes writes zero rows. No duplicates when an email changes.

### 7.2 Assistant Telegram account (P0)

- Connect the new SIM account with the existing phone + code + 2FA flow.
- The account belongs to the workspace, not to a user (G5).
- Sync filter: store a dialog only if the peer matches a synced sister (by Telegram user ID, username or phone). Drop all other dialogs. Do not sync group history, except the groups in 7.8.
- Change the current chat sync from "create a contact per chat" to "match only, never create". Turn off group-member import for this workspace.
- Sync interval: every 15 min for this account (today: once a day at 03:00 UTC), so replies show the same day.
- Operators can also use the same account on their phones. Telegram allows parallel sessions.
- **Ops note (not code):** keep the SIM alive with a 1 UAH/month Monobank top-up.

### 7.3 Contact provisioning (P0 Telegram, P1 Google)

- Trigger: a new sister record, or a first value in `Date of Nama` / `Date of Initiation`.
- Telegram: import the sister into the assistant account's contacts (`contacts.importContacts` with phone; resolve `@username` to a user ID). Display name: `First Last (Sanskrit Name)`. Store `telegram_user_id`.
- Google: create or update the contact in the label "Sisters" in a configured Google account. Write only contacts that PingCRM created. Never read, change or delete other contacts. Needs the `contacts` scope (today: `contacts.readonly`).
- Which Google account: open question Q1. Didi's Google data is mixed with the Amurtel account today.
- **Acceptance:** a new record with a phone number shows in the assistant's Telegram contacts in less than 1 h.

### 7.4 Message templates (P0)

- Import the 13 Airtable templates as seeds.
- One template has one variant per language. The sister's `preferred_language` selects the variant. No variant → the row shows a warning; the operator selects a fallback language or skips.
- Placeholders: `{first_name}`, `{sanskrit_name}`, `{city}`, `{country}`, `{lesson_received}`, `{next_lesson}`, `{event_name}`, `{event_date}`, `{event_city}`, `{event_link}`.
- A placeholder with no value blocks that row until the operator edits it.
- Audience rule per template: minimum and maximum `Lesson Received`. Example: "5th lesson prep" is only for sisters with the 4th lesson. The batch UI does not allow a send outside the rule.
- AI draft (P1): the existing Claude integration writes a variant in the sister's language from Didi's English text. The operator reviews every draft.

### 7.5 Segments and batch send (P0)

- Filters: country, city, preferred language, `Lesson Received`, `Init - Margii Stage`, `Nama to Init stage`, `Status`, age range, days since last lesson, marital status, source, project, has Telegram, "no message for N days", birthday / anniversary window.
- Save a filter as a named segment ("Germany · 1st lesson").
- Batch flow: segment → template → preview list (one rendered message per sister) → edit or remove rows → **Send**.
- Default exclusions: `Do Not Contact`, no resolvable Telegram peer, same template sent in the last 30 days, `archived`.
- Send by `telegram_user_id` first, then `telegram_username`, then phone (via contact import). Today the send path needs a username.
- Send queue (Celery):
  - Random delay of 20–90 s between messages.
  - Daily cap per account. Default: 20 messages to peers with no previous dialog, 100 in total. The admin can change it.
  - On `FloodWait`, wait the given time. On `PeerFlood`, stop the batch and alert the admin.
- Per-message state: `queued`, `sent`, `failed` (reason), `replied`.
- Each sent message becomes an interaction on the sister's timeline, with the template ID and the operator.
- **Acceptance:** a 50-sister batch is ready to send in less than 5 min of operator time. A queue restart does not send a message twice.

### 7.6 Events and calendar reminders (P0)

- Source: one Google Calendar that Didi (or the assistant) keeps for tours and retreats. An AI agent can also create the events.
- Reuse the Google OAuth flow and the `calendar.readonly` scope. Do not reuse the attendee-to-contact logic.
- Each calendar event maps to a PingCRM event: name, start, end, city, country, landing link, languages. Location parsing: `City, Country` in the location field; the operator can correct it in PingCRM.
- Reminder offsets per event type. Default T-30, T-14, T-7 (session request). The retreat type can use T-60, T-30, T-7 (the current Airtable rule).
- At each offset the system creates a **batch draft**: segment = sisters in the event country (or city), template = the event reminder template, plus a line about the next lesson review.
- The assistant gets a notification: "Didi arrives in Germany in 30 days. 161 sisters. Batch ready."
- No send without operator approval (non-goal: automatic sends).
- **Acceptance:** a new calendar event creates 3 scheduled drafts. Each draft appears on its day. The 100-record limit of Airtable does not apply.

### 7.7 Stage cadence rules (P1)

- Rule = trigger + delay + template + audience. Examples from the session and PRD-001:
  - `Lesson Received = 1st` and 30 days after `Date of Lesson Received` → "DC prep".
  - 60 days after 1st lesson → "2nd lesson prep".
  - Nama and 14 days after `Date of Nama` → "Nama check-in after retreat".
- A rule creates a daily batch draft, not a send.

### 7.8 Telegram groups and channels (P1)

- Two spaces: one channel + linked discussion group for Nama Mantra sisters, one for all sisters.
- After `Date of Nama` is set, add the sister's row to a batch that sends the invite link. Direct add only when her privacy settings allow it; otherwise the link.
- Store group membership per sister.

### 7.9 Inbox and "Needs Didi" flag (P1)

- Messages from sisters to the assistant account show on the timeline with an unread state.
- The assistant can flag a thread "Needs Didi" with a short note. Didi sees a list of flagged threads and replies from Didi's own phone.

### 7.10 Airtable quick wins (P2)

- **Romanisation:** PingCRM writes Latin versions of `First name (Input)`, `Last name (Input)` and `City (Input)` into the empty `…Romanised` fields. Needs write scope on those 3 fields only.
- **Orphan lesson requests:** list `Higher Lessons Requests` rows whose `Clean Email` matches no sister (for example sisters initiated by another Acharya). Link each to the Airtable record so Didi can fix it.

## 8. Non-functional requirements

### 8.1 Privacy

- Store sister data only (D4). Log every drop of a non-sister dialog as a count, not as content.
- Encrypt Telegram sessions, Google tokens and the Airtable token at rest.
- Audit log: who sent what to whom, and when.
- Self-hosted. No sister data leaves the server, except the LLM call for AI drafts (P1), which sends the draft text and the first name only.

### 8.2 Telegram account safety

- A fresh account that sends to many strangers gets restricted by Telegram. This is the main MVP risk (section 11).
- Warm-up plan: first 2 weeks, cap at 10 new peers per day; sisters get the assistant's number or `t.me` link at events and write first.

### 8.3 Reliability

- The send queue survives a restart. Idempotency key per (batch, sister).
- Sync errors and send errors show in the UI and in the log with context.

### 8.4 Workspace model

- One workspace per Acharya. Users join a workspace with a role (section 3). No cross-workspace reads.
- This is the "apartment in a building" model (D9). It is the base for other Acharyas in [PRD-003](003-forms-module-and-module-system.md).
- Today every row and every query uses `user_id` (section 4.2). Two ways to add workspaces:

| Option | How | Cost | Risk |
| --- | --- | --- | --- |
| **A. Owner-as-workspace (recommended for MVP)** | Keep the existing `users` row as the data owner (the workspace). Add `workspace_members (owner_user_id, member_user_id, role)`. A new dependency resolves the logged-in member to the owner's `user_id`; queries do not change. Role checks guard sensitive fields and settings. | Small: one table, one dependency, role guards | Audit fields ("who sent") need the member ID explicitly |
| B. Full `workspace_id` refactor | New `workspaces` table; `workspace_id` on every table; re-scope every query and MCP tool | Large: ~280 `user_id` filter sites in `app/` and `mcp_server/`, 179 uses of `current_user.id` | Merge conflicts with upstream |

- Record the acting member on every send, note and flag.

### 8.5 Language

- Message content: every language in section 4.1.
- UI: English in the MVP. Ukrainian UI depends on Q5.

## 9. Success metrics

| Metric | Baseline (2026-09-26) | Target 90 days after launch |
| --- | --- | --- |
| Sisters with a Telegram message from the assistant in the last 90 days | not measured | 60 % of sisters with a Telegram ID |
| `Status = Unknown` | 201 | less than 100 |
| Sisters in the assistant's Telegram contacts | 0 | all sisters with a phone or username |
| Events with all 3 reminder batches sent | 0 (automations OFF) | 100 % of calendar events |
| Telegram account restrictions | – | 0 |

## 10. Milestones

```
M0  Set-up         SIM + Telegram account, workspace + roles (option A),
                   module skeleton (PRD-003 section 6), deploy
M1  Read           Airtable sync, sister profile view, segments          (7.1, 7.5 filters)
M2  Send           Templates, batch preview, send queue, timeline        (7.4, 7.5)
M3  Provision      Telegram contacts import, Google "Sisters" label      (7.3)
M4  Calendar       Google Calendar events -> reminder drafts             (7.6)
M5  Care loop      Cadence rules, groups/channels, inbox + Needs Didi   (7.7-7.9)
M6  Airtable fixes Romanisation, orphan requests                         (7.10)
```

## 11. Risks

| Risk | Level | Mitigation |
| --- | --- | --- |
| Telegram restricts the new account for bulk messages (`PeerFlood`, spam reports) | **High** | Warm-up, low daily caps, random delays, human review, sisters write first, stop on the first `PeerFlood` |
| Airtable field renames break the sync | Medium | Map by field ID; show sync errors in the UI |
| Two systems drift | Medium | Airtable owns initiation fields; PingCRM owns messages, events and segments only |
| Email as key in Airtable loses lesson requests | Medium | PingCRM keys on record ID; orphan list (7.10) |
| Google data still mixed with Amurtel | Medium | Wait for Didi's own account (Q1); Google parts are P1 |
| A stolen Telegram session gives full account access | Medium | Encrypt at rest, admin-only session management, revoke from Telegram "Devices" |
| iCloud contacts clean-up not finished | Low | MVP targets the assistant's account, not Didi's phone |

## 12. Open questions

| # | Question | Owner |
| --- | --- | --- |
| Q1 | Which Google account and calendar are the source for events and "Sisters" contacts, after the split from Amurtel? | Didi |
| Q2 | Reminder offsets: T-30 / T-14 / T-7 (session) or T-60 / T-30 / T-7 (Airtable)? Different per event type? | Didi |
| Q3 | Channel language: one English channel, or one per language? | Didi |
| Q4 | Later: should Didi's own account connect in send-only mode (no history sync)? | Didi, Taras |
| Q5 | Does Madhu need a Ukrainian or Russian UI? | Madhu |
| Q6 | Sisters from other Acharyas who request a higher lesson review: create a sister record, or keep them in a separate list? | Didi |
| Q7 | When is WhatsApp needed for Romania? | Didi |
