# PRD: Initiate Support CRM (PingCRM for Acharyas)

## 1. Executive Summary & Vision

### 1.1 Context & Problem
During spiritual outreach and tours (such as the *Tochka Sily* tour across Ukraine), Acharya Didiananda Deva Priya initiates dozens of spiritual seekers (e.g., 47 Nama Mantra initiations in 10 days). However, without systematic follow-up, initiates quickly drift away—analogous to pouring "water into the sand."

Simultaneously, direct 1-to-1 communication with the Acharya creates a severe operational bottleneck:
- Initiates often feel shy or hesitant to disturb a busy teacher with simple everyday questions.
- The Acharya is flooded with direct messages across multiple personal messengers; opening a message while busy leads to it being buried or forgotten.
- Generic sales CRMs (HubSpot, Close, etc.) operate on commercial pipelines (leads, deals, pipeline stages) that feel robotic, transactional, and antithetical to the warmth, sisterhood, and sacred trust of spiritual discipleship.

### 1.2 Vision
Transform **PingCRM** into the **Initiate Support CRM**—a warm, relationship-centric operating system for spiritual teachers (Acharyas) and their dedicated assistants (Local Full-Time / Local Part-Time workers, LFTs/LPTs).

The system combines:
1. **Multi-Account Telegram Relaying**: Seamless coexistence of the Assistant's frontline account and the Acharya's personal account under a single, unified contact timeline.
2. **Acharya Triage & Escalation**: Routine care, resources, and check-ins are handled by the Assistant, while deep spiritual, personal, or philosophical questions are routed to a dedicated "Acharya Review Queue" for weekly audio/text replies.
3. **Initiate Lifecycle & Sadhana Milestones**: Tracking stages from Nama Mantra through Lesson 1 up to Lesson 6, with automated milestone check-ins (e.g., 45-60 days post-Lesson 1: "Are you ready for Lesson 2?").
4. **Content Tiering**: Guardrails ensuring beginner contacts (Nama Mantra) receive accessible, universal mindfulness content, while advanced Margiis receive deeper philosophy and Baba stories without accidental cross-exposure.
5. **Human-in-the-Loop AI Copilot**: Drafting warm, multilingual follow-ups (Ukrainian, Romanian, Russian, English) for the Assistant to review and send with one click.
6. **Open-Source Acharya Blueprint**: A reusable, exportable template and Standard Operating Procedure (SOP) that other Acharyas can deploy in 15 minutes.

---

## 2. Personas & Roles

| Role | User | Primary Channel | Core Needs |
| :--- | :--- | :--- | :--- |
| **Acharya** | Didiananda Deva Priya | Telegram (personal), Web CRM | Quiet focus. Wants an escalation inbox for questions needing deep guidance. Does not want unread messages lost in personal chat feeds. |
| **Assistant / LPT** | Madhu (LPT) | Dedicated Telegram SIM (`Madhu | Assistant of Didi`) | Frontline sisterly care, sending check-ins, answering FAQ, sharing Dharmacakra links, escalating complex questions to Didi. |
| **Initiate / Sister** | Spiritual seeker | Telegram / WhatsApp | Feels remembered, loved, and guided; has an accessible sister (Madhu) to ask daily practice questions, with direct access to Didi when needed. |
| **Local Mentors** | Dharmadeha / Local Sanghas | Telegram | Connected with new initiates in their specific city (e.g., Vinnytsia, Cherkasy, Mykolaiv, Berlin, Bucharest). |

---

## 3. Core Epics & Functional Requirements

### Epic 1: Multi-Account Telegram Integration & Shared Inbox
*Current State:* PingCRM supports a single Telegram MTProto session per user (`user.telegram_session`).
*Target State:* PingCRM supports multiple Telegram accounts feeding into a shared organization workspace.

- **1.1 Multiple Telegram Accounts**:
  - Connect `Account 1: Madhu (Assistant)` (primary frontline communicator via a dedicated phone number/SIM).
  - Connect `Account 2: Didiananda (Acharya)` (senior escalation communicator).
- **1.2 Unified Contact Timeline**:
  - Inbound messages from an initiate to either account appear in that contact's central thread.
  - Outbound messages display sender badges: `[Sent by Madhu]` or `[Sent by Didi]`.
- **1.3 Escalation & Triage Queue**:
  - Assistant can flag any incoming message or thread as `Escalated to Acharya` with an optional private note.
  - Acharya has a focused UI view (`/triage` or filtered inbox) showing only escalated conversations.
  - Acharya can record a voice note or type a response directly in the CRM, which sends through the Acharya's Telegram session to the initiate.

### Epic 2: Initiate Domain Model & Custom Metadata
*Current State:* Generic CRM contact model with social handles and tags.
*Target State:* Specialized spiritual mentorship fields.

- **2.1 Initiation Milestones**:
  - `initiation_tier`: Enum (`nama_mantra`, `lesson_1`, `lesson_2`, `lesson_3`, `lesson_4`, `lesson_5`, `lesson_6`, `acarya`).
  - `current_lesson_date`: Date when the current lesson was given.
  - `days_in_current_stage`: Computed field for automated cadence triggers.
- **2.2 Spiritual Practice Health**:
  - `sadhana_status`: Enum (`active`, `struggling`, `dormant`, `vip_leader_candidate`).
  - `last_sadhana_checkin`: Timestamp of last contact regarding practice.
- **2.3 Geo, Sangha & Mentorship**:
  - `city`, `country`.
  - `local_sangha_id`: Reference to local Dharmacakra group.
  - `peer_mentor_id`: Link to an assigned senior sister / Dharmadeha coach.
- **2.4 Language & Channel**:
  - `preferred_language`: `uk` (Ukrainian), `ro` (Romanian), `en` (English), `ru` (Russian).
  - `primary_platform`: `telegram` (default for UA/EU) or `whatsapp` (critical for Romania).

### Epic 3: Safe Broadcasts & Content Tiering
*Problem:* Sharing advanced devotional stories with Nama Mantra initiates can confuse or alienate them. Sharing basic stress-relief guides with advanced margiis is redundant.

- **3.1 Audience Tiers**:
  - `Tier 0 (Nama Mantra)`: Universal wellness, stress management, focus, 7-day challenge follow-up.
  - `Tier 1 (1st Lesson / Margiis)`: Dharmacakra schedules, Kiirtan recordings, foundational philosophy, Didi interactive webinars.
  - `Tier 2 (Higher Lessons / Workers)`: Baba stories, microvita, spiritual treatises, LFT/WT retreat announcements.
- **3.2 Content Gatekeeper**:
  - Broadcast engine enforces audience matching: broadcasts tagged with `Tier 2` cannot be sent to contacts with `Tier 0` or `Tier 1`.

### Epic 4: Automated Care Workflows & AI Copilot

- **4.1 Workflow: Post-Nama Mantra Onboarding (7-Day Bot Handoff)**:
  - Event attendees join the 7-day challenge via QR code.
  - On Day 7, a webhook notifies PingCRM: "7-Day Challenge Completed".
  - Generates a task for Madhu: "Send warm congratulations and introduce local sister in [City]".
- **4.2 Workflow: Lesson 1 Follow-up Cadence**:
  - **Day +3**: Send welcome packet: how to create a meditation space at home + link to Dharmacakra onboarding video.
  - **Day +30**: First practice check-in: "How is your concentration and daily routine going?".
  - **Day +60**: Second lesson readiness check: "Didi is hosting online sessions / visiting soon; feel free to discuss 2nd lesson readiness".
- **4.3 Workflow: Tour & Geo-Targeted Outreach**:
  - When Didi plans a trip to a city/country (e.g. Cherkasy or Germany), the system filters all contacts in that region and drafts personalized invitations for Madhu to approve with one click.
- **4.4 Human-in-the-Loop AI Assistant**:
  - Drafts responses in the initiate's preferred language (e.g., translating Didi's English guidance into warm Ukrainian or Romanian).
  - Madhu reviews, tweaks, and sends; zero automated unreviewed "bot spam".

### Epic 5: Airtable Importer & Acharya Blueprint (SOP)
- **5.1 Airtable Import Utility**:
  - Importer script/UI to map existing Airtable initiation columns directly into PingCRM initiate records.
- **5.2 Open-Source SOP Documentation**:
  - Detailed, non-technical guide for newly posted Acharyas to set up this system for their dioceses/sectors in 15 minutes.

---

## 4. Technical Architecture & Database Changes

### 4.1 Schema Additions (SQLAlchemy / Alembic)

```python
# New model: TelegramAccount (1-to-many relationship with User/Workspace)
class TelegramAccount(Base):
    __tablename__ = "telegram_accounts"
    id = Column(UUID, primary_key=True, default=uuid4)
    user_id = Column(UUID, ForeignKey("users.id", ondelete="CASCADE"))
    account_label = Column(String(50))  # e.g., "Madhu (Assistant)", "Didi (Acharya)"
    phone_number = Column(String(30))
    session_string = Column(Text)       # Encrypted Telethon session
    is_active = Column(Boolean, default=True)
    is_primary_frontline = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), default=func.now())

# New model: InitiateProfile (extension of Contact)
class InitiateProfile(Base):
    __tablename__ = "initiate_profiles"
    id = Column(UUID, primary_key=True, default=uuid4)
    contact_id = Column(UUID, ForeignKey("contacts.id", ondelete="CASCADE"), unique=True)
    initiation_tier = Column(String(30), default="nama_mantra")  # nama_mantra, lesson_1..6
    current_lesson_date = Column(Date, nullable=True)
    sadhana_status = Column(String(30), default="active")        # active, struggling, dormant
    city = Column(String(100), nullable=True)
    country = Column(String(100), nullable=True)
    preferred_language = Column(String(10), default="uk")       # uk, ro, en, ru
    preferred_messenger = Column(String(20), default="telegram")
    local_mentor_name = Column(String(100), nullable=True)
    needs_acharya_attention = Column(Boolean, default=False)
    acharya_escalation_note = Column(Text, nullable=True)
    last_checkin_at = Column(DateTime(timezone=True), nullable=True)
```

### 4.2 Backend & Worker Architecture
- **FastAPI**: Endpoints for managing multiple `telegram_accounts`, initiate profile CRUD, and triage queue (`/api/v1/triage`).
- **Telethon Client Manager**: Multi-client singleton pool maintaining background MTProto event listeners for each active `TelegramAccount`.
- **Celery Beat**: Daily scheduled tasks evaluating lesson intervals (Day 3, 30, 60) and generating action cards in the CRM.

---

## 5. Implementation Roadmap

- [ ] **Milestone 1**: Database migrations for `TelegramAccount` and `InitiateProfile`.
- [ ] **Milestone 2**: Multi-session Telethon manager (support sending & receiving from multiple accounts).
- [ ] **Milestone 3**: UI overhaul of Contact Details (Initiate badges, lesson milestones, sister notes).
- [ ] **Milestone 4**: Acharya Escalation Inbox (`/triage`) with voice-reply capability.
- [ ] **Milestone 5**: Celery Beat care triggers (Nama Mantra 7-day, Lesson 1 cadence, Geo-pings).
- [ ] **Milestone 6**: Airtable import script and SOP blueprint guide for other Acharyas.
