# PRD-003: Forms Module and Module System

| | |
| --- | --- |
| **Status** | Draft |
| **Date** | 2026-09-26 |
| **Product owner** | Didi Ananda Devapriya (Acharya) |
| **Build owner** | Taras Kornichuk |
| **Depends on** | [PRD-002](002-sister-followup-telegram.md) (workspace roles, Telegram account, templates) |
| **Takes over** | PRD-001 Epic 5.2 (blueprint for other Acharyas) |

## Sources

| Source | What it gave |
| --- | --- |
| Working session, 2026-09-26 (Svidok #1306) | Separate forms per request type, email-key problem, broken transliteration, event forms, "apartment in a building", AI agents customise the system |
| Airtable base "Didi's Initiation Database", read 2026-09-26 | Form targets, automations, the `Higher Lessons Requests` table |
| `Acarya Airtable CRM Process Documentation.docx` | Form SOPs, "duplicate the base" SOP for other Acharyas |
| `github.com/formsmd/formsmd`, read 2026-09-26 | Candidate form renderer |
| PingCRM code at `main@7537ca9` | Current extension points |

---

## 1. Problem

### 1.1 Forms

Forms are the start of every sister's record. Today they live in Airtable:

```
New Initiation form ........ EN, UK, RU, RO, IT   (5 forms)  -> Sisters
Higher Lesson Request form . EN, UK, RU, RO, IT   (5 forms)  -> Higher Lessons Requests -> match by email -> Sisters
Intake Form Other Reviews ..                      (1 form)   -> Sisters
Student Status Update ...... country leaders      (1 form)   -> Country Leaders update -> Sisters
Cyrillic converter ......... 6 automations (RU, UK x 3 form types), each runs a script on submit
```

| Pain | Evidence |
| --- | --- |
| One form per language | At least 12 Airtable forms to keep in step. A field change means 5 edits. |
| Email is the only key | A request with a new or wrong email matches nothing and does not show in any interface. Sisters from another Acharya cannot use the Higher Lesson form at all. |
| Transliteration breaks | The Cyrillic converter did not run on submissions from 2026-09-26. |
| Event and Prachar forms are scattered | Instagram forms, Google Forms, paper at events. Tour RDS data (event, place, attendance) sits in Google Sheets. |
| Forms end at "saved" | After a submit, nobody gets a task and the sister gets no Telegram contact. |

### 1.2 Open-source network

Didi and Taras plan to offer the system to other Acharyas for self-hosting. Each Acharya will customise it with AI agents. Today:

- The Airtable "duplicate the base" SOP copies structure only, and automations fail on free plans.
- In PingCRM, every integration is hard-wired: 20 routers in `backend/app/main.py:106-125`, one static Celery beat table in `backend/app/core/celery_app.py:24-95`, one settings card per platform in `frontend/src/app/settings/_components/platform-cards/`.
- There is no place for an Acharya-specific feature. An agent that adds one edits core files. Didi saw this with AI tools: "one small tweak … redoes everything and loses other things that were working".

## 2. Goals and non-goals

### Goals

| ID | Goal | Target |
| --- | --- | --- |
| F1 | One form definition, many languages | 1 form + 5 translations replaces 5 Airtable forms |
| F2 | Typeform-like public forms | One question per screen, mobile first, QR code for events |
| F3 | Every submission lands | 0 lost submissions. No match → review queue, never a silent drop. |
| F4 | A submission starts a flow | Contact created or matched, request created, Telegram contact provisioned, assistant notified |
| F5 | Owners and agents can build forms | A form is a text spec that a person or an AI agent can edit, with live preview |
| M1 | Features live in modules | An Acharya install can turn a module on or off. An agent can change one module and not touch core. |
| M2 | Core stays close to upstream | Security fixes from `sneg55/pingcrm` merge with few conflicts |

### Non-goals

- Drag-and-drop visual builder (P2, section 7).
- Payment forms.
- Third-party runtime plugins or a plugin marketplace (section 6.1).
- Replacing Airtable before PRD-002 ships.

## 3. Users

| User | Forms need |
| --- | --- |
| Acharya (owner) | Create and change forms; see all submissions; share a link or QR at a retreat |
| Assistant | Review the match queue; process new requests |
| Country leader | Status update form for the leader's region only (later) |
| Sister / seeker | Fill a short form in her language on a phone |
| AI agent (for an Acharya) | Create or edit a form spec from a voice request; open a change for review |
| Other Acharya (self-hoster) | Install, turn on the needed modules, keep the data private |

## 4. Form engine evaluation

### 4.1 formsmd

| Check | Finding |
| --- | --- |
| License | Apache-2.0. Compatible with PingCRM's AGPL-3.0; we can copy code with attribution. The hosted product sells watermark removal ($99 / $299); the library has no such limit. |
| Maturity | 782 stars, 1 maintainer, last commit 2025-04-27 (17 months ago), `formsmd` on npm at 73 downloads/week, open issues on multi-file upload and payments since 2024 |
| Model | Client-side JS. Form = Markdown-like text or a JS `Composer` API. `postUrl` sends JSON to our endpoint. `jumpCondition` / `displayCondition` for branching. One question per slide. Resume from `localStorage`. |
| i18n | UI strings for en, ar, bn, de, es, fr, ja, pt, zh. **No uk, ru, ro, it.** |
| React / Next.js | No React wrapper. Uses `window` and `document`; must load client-side only. |
| Builder | None. Code or Markdown only. |

**Verdict:** do not depend on it. Take its ideas: text-first form specs, one question per screen, branching by condition, `postUrl` to our own API. Its Apache-2.0 code and CSS can be copied where useful.

### 4.2 Options

| Option | Summary | For | Against |
| --- | --- | --- | --- |
| A. Embed formsmd | Load the bundle on public pages | Fast Typeform look | Unmaintained; no uk/ru/ro/it; client-only; no builder |
| **B. Own engine (recommended)** | Text spec (TOML) → validated schema (Pydantic + JSON Schema) → own React one-question-per-screen renderer | Full control of languages, SSR, data mapping; agent-editable specs; no new service | We build the renderer and a basic editor |
| C. Formbricks as a sidecar | Separate Next.js + Prisma service; webhook into PingCRM | Visual builder today; active project; AGPL-3.0 matches ours | Second app, second database, second login; data mapping lives outside the CRM; enterprise parts under a separate licence |

Option B fits the goals: the spec is text (F5, agents), the mapping to CRM fields is ours (F3, F4), and it runs inside the module system (M1).

## 5. Functional requirements: forms module

### 5.1 Form spec (P0)

- One form = one TOML spec, stored versioned in Postgres. Export and import as a `.toml` file.
- TOML because people and LLM agents both read and edit it well.
- Parts: form meta, languages, steps, fields, conditions, field-to-CRM mapping, after-submit actions.
- Every visible string has a value per language. A missing translation fails validation.

```toml
[form]
slug = "higher-lesson-request"
languages = ["en", "uk", "ru", "ro", "it"]
audience = "sister"              # sister | seeker | leader

[[fields]]
id = "email"
type = "email"
required = true
maps_to = "contact.email"
label.en = "Your email"
label.uk = "Ваша електронна пошта"

[[fields]]
id = "lesson_requested"
type = "choice"
choices = ["1st", "2nd", "3rd", "4th", "5th", "6th"]
maps_to = "request.lesson_requested"
label.en = "Which lesson do you ask for?"
label.uk = "Який урок ви просите?"

[[fields]]
id = "first_acharya"
type = "text"
show_if = "is_new_contact"
maps_to = "request.initiated_by"
label.en = "Who gave you your first lesson?"
label.uk = "Хто дав вам перший урок?"

[[actions]]
on = "submit"
do = ["match_contact", "create_request", "notify_assistant", "offer_telegram_link"]
```

### 5.2 Public renderer (P0)

- URL: `/f/{workspace}/{slug}?lang=uk`. Language from `?lang`, then browser language, then the form default. A language switch on every screen.
- One question per screen, progress bar, back button, keyboard and touch input.
- Server-side render for first paint; works on slow mobile networks.
- Prefill from URL parameters and hidden fields (for example `source=tochka-syly-2026-09`).
- QR code per form and per source value, for retreats and events.
- Spam guard: honeypot field + rate limit per IP. No third-party captcha by default.
- Footer link to the source code (AGPL-3.0 section 13, see section 6.4).

### 5.3 Submission pipeline (P0)

```
submit
  -> store raw submission (immutable, with form version + language)
  -> transliterate Cyrillic name/city fields to Latin (server side)
  -> match contact: email, then phone, then Telegram username, then name + birthday
       one match ........ link
       no match ......... new contact (if the form allows) or match queue
       2+ matches ....... match queue
  -> create request (lesson request, review, event registration, status update)
  -> run actions (notify assistant, provision Telegram contact, add to pipeline)
  -> thank-you screen with a t.me link to the assistant account
```

- The raw submission stays even when the match or an action fails. Nothing is lost (F3).
- The thank-you screen asks the sister to open a chat with the assistant. When she writes first, later messages from the new account carry less spam risk (PRD-002 section 8.2).
- Match queue: the assistant sees each unmatched submission with the best candidates and links it with one click.

### 5.4 Builder (P1)

- Text editor for the TOML spec with live preview on the right, validation errors inline.
- "Translate missing strings" button: Claude drafts the missing languages; the owner reviews them.
- Version history; publish creates a new version; old submissions keep their version.
- An AI agent can open a draft version through the API or MCP; the owner publishes it.

### 5.5 Built-in form set (P0 seeds)

| Form | Replaces | Target |
| --- | --- | --- |
| New initiation request | 5 Airtable forms | contact + initiation request (Nama or 1st lesson) |
| Higher lesson request | 5 Airtable forms + email-only match | lesson request; accepts sisters from other Acharyas |
| Other reviews intake | 1 Airtable form | review request |
| Status update (country leader) | 1 Airtable form + quarterly email | status change, limited to the leader's region |
| Event registration (P1) | Google Forms, paper | event attendance for RDS; seekers do not become sisters |

### 5.6 Coexistence with Airtable (P0 while Airtable is the system of record)

- PRD-002 keeps Airtable as the source for initiation data (decision D1).
- Until that changes, the forms module writes each accepted submission to Airtable through an **Airtable adapter** (create or update the `Sisters` / `Higher Lessons Requests` record, fill the `…Romanised` fields). Didi's interfaces and RDS dashboard keep working.
- When the initiation data moves to PingCRM, turn the adapter off. No form changes.

## 6. Module system

### 6.1 Decision: a module boundary, not a plugin runtime

| Question | Answer |
| --- | --- |
| Do we need a place for features outside core? | **Yes.** Acharyas need different parts; agents need a small, safe area to change; core must stay mergeable with upstream. |
| Do we need runtime plugins (install third-party code into a running server, marketplace, sandbox)? | **No, not now.** Few installs, a small team, and sister data in every module. Third-party code with access to that data is a privacy risk we cannot review. Python entry points can come later with no rework if the contract below is stable. |

Modules are in-repo Python and TypeScript packages. The install turns them on in config. Agents change them in a fork, with tests.

### 6.2 Module contract

```
backend/app/modules/<name>/
  module.toml        name, version, depends_on, settings schema, roles, beat entries
  models.py          SQLAlchemy models (tables prefixed <name>_)
  api.py             FastAPI router, mounted at /api/v1/m/<name>
  tasks.py           Celery tasks; registered through the module loader
  events.py          handlers for core events
  mcp.py             MCP tools this module exposes (optional)
  migrations/        Alembic version location for this module
  tests/
  AGENTS.md          what an agent may change here, and how to test it

frontend/src/modules/<name>/
  index.ts           nav items, settings panel, contact-detail panels
  pages/             rendered through /m/<name>/[...path]
```

- **Loader:** `MODULES=airtable_sync,telegram_outbound,forms` in env. At start-up the loader mounts routers, registers tasks and beat entries, and adds nav items. This replaces the manual step in `CLAUDE.md` ("register new Celery tasks in `app/services/tasks.py`") for module tasks.
- **Core events:** `contact.created`, `contact.updated`, `interaction.created`, `submission.received`, `lesson.given`, `event.upcoming`. Modules react to events; they do not import each other's internals.
- **Data scope:** every module table carries the workspace owner key (PRD-002 section 8.4). The loader refuses a module that queries without it (lint rule + test).
- **Isolation of failure:** a module that fails to load is logged and switched off; core still starts.

### 6.3 Module map

| Module | Source | For Didi | Default for a new Acharya install |
| --- | --- | --- | --- |
| `core` (contacts, timeline, auth, workspace, roles) | existing | on | on |
| `telegram` (session, sync, send) | existing, refactor | on | on |
| `airtable_sync` | PRD-002 7.1 | on | off |
| `outreach` (segments, templates, batch send, cadence) | PRD-002 7.4–7.7 | on | on |
| `events` (calendar, reminders, attendance) | PRD-002 7.6 | on | on |
| `forms` | this PRD | on | on |
| `donations`, `rds_reports` | later, from Airtable | off | off |
| `twitter`, `linkedin`, `meta`, `apollo`, `gmail`, `whatsapp`, `extension` | existing upstream code | off | off |

Move existing integrations into modules only when we touch them. First reference module: `airtable_sync` (new code, no upstream conflict).

### 6.4 Open-source packaging for other Acharyas

- **Licence:** PingCRM is AGPL-3.0. Section 13 applies when users interact with the software over a network. Public forms make sisters such users, so each install must offer its source: the footer link in 5.2 does this. Each Acharya's fork stays AGPL-3.0.
- **Install:** reuse `AGENT_SETUP.md` (already an agent-run runbook for self-hosting). Add an "Acharya profile" step: turn on the modules in the table above, load the seed forms and templates, create the owner and assistant.
- **Own your data:** full export of a workspace (contacts, submissions, messages, templates, forms) as CSV + JSON at any time. This answers the fear of data "disappearing" and of lock-in to a system outside the Acharya's control (PRD-002 D8).
- **Integration points, not one central database:** each install exposes a documented API and outgoing webhooks. Two Acharyas who agree can connect them later (for example a higher lesson review for a sister from another Acharya). No shared database.
- **Agent-first:** each module ships `AGENTS.md`, tests, and MCP tools. The SOP for a new Acharya is: "tell your agent what you need; the agent changes one module and runs its tests".

## 7. Milestones

```
F0  Module loader + contract, first module airtable_sync (with PRD-002 M0-M1)
F1  Forms core: TOML spec, validation, renderer, raw submissions, transliteration
F2  Pipeline: contact match + queue, requests, actions, Airtable adapter
F3  Seed forms at parity with Airtable (5 languages), QR, t.me thank-you link
F4  Builder: text editor + preview, AI translation, versions, agent drafts via MCP
F5  Event registration + attendance for RDS
F6  Packaging: Acharya profile in AGENT_SETUP.md, workspace export
P2  Drag-and-drop builder, country-leader portal, payments
```

## 8. Success metrics

| Metric | Baseline | Target |
| --- | --- | --- |
| Lost submissions (no record anywhere) | unknown, reported in the session | 0 |
| Forms to edit for one field change | 5 | 1 |
| Submissions with a Latin name | converter fails on some | 100 % |
| Time from submit to assistant notification | none | less than 1 min |
| New install to first public form (agent-run) | – | less than 1 h |

## 9. Risks

| Risk | Level | Mitigation |
| --- | --- | --- |
| We build a renderer and a builder ourselves | Medium | Text editor first; visual builder is P2; copy formsmd ideas and Apache-2.0 code |
| Airtable adapter and Airtable automations fight (double updates) | Medium | Adapter writes the same fields the Airtable forms write; turn off the matching Airtable automation per form at cut-over |
| Module boundary slows the PRD-002 MVP | Low | Loader is small; start with one module |
| Agents in forks break the module contract | Medium | Contract tests in CI; loader refuses invalid `module.toml` |
| AGPL obligations missed by an Acharya install | Low | Source link built into the public form footer |
| Public forms attract spam | Low | Honeypot, rate limit, match queue |

## 10. Open questions

| # | Question | Owner |
| --- | --- | --- |
| Q1 | Should seekers from event forms get a contact record in the Acharya's workspace, or only an attendance count? (Session: keep cold leads out.) | Didi |
| Q2 | Which fields can a country leader see and change? | Didi |
| Q3 | Do we keep the per-language form links for existing QR codes and printed material? | Didi |
| Q4 | When does initiation data move from Airtable to PingCRM (the end of the adapter)? | Didi, Taras |
| Q5 | Which other Acharyas pilot the install, and when? | Didi |
