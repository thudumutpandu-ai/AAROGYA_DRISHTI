# Aarogya Drishti

**An AI-assisted oral cancer early-detection, telemedicine and patient-care platform for remote tribal communities.**

> Built for the **AAROHAN Inter-Centre Hackathon 2026**, organised by ITDA Paderu, Alluri Sitharama Raju District, Andhra Pradesh (27 Oct - 4 Nov 2026).
> Theme: *AI for Public Health | Technology for Tribal Communities | From Screening to Treatment*
> Format: open-book developer hackathon. Open-source tools, documentation and AI coding assistants are allowed, with disclosure (see [section 23](#23-team-hackathon-rules-and-submission)).

> ⚠️ **NOT FOR CLINICAL USE.** This is a hackathon demonstration. It uses **synthetic patients and licensed demo images only**. The AI component is a research/mock adapter and must never be used to diagnose, stage or rule out cancer.

---

## Table of contents

1. [Why this project exists](#1-why-this-project-exists)
2. [What it does](#2-what-it-does)
3. [Non-negotiable safety rules](#3-non-negotiable-safety-rules)
4. [End-to-end workflow](#4-end-to-end-workflow)
5. [Users and permissions](#5-users-and-permissions)
6. [Architecture](#6-architecture)
7. [Tech stack](#7-tech-stack)
8. [Repository structure](#8-repository-structure)
9. [Getting started](#9-getting-started)
10. [Configuration](#10-configuration)
11. [Data model](#11-data-model)
12. [Case state machine](#12-case-state-machine)
13. [API overview](#13-api-overview)
14. [Image quality and AI pipeline](#14-image-quality-and-ai-pipeline)
15. [Telugu and offline-first design](#15-telugu-and-offline-first-design)
16. [Security and privacy](#16-security-and-privacy)
17. [Testing](#17-testing)
18. [Deployment and recovery](#18-deployment-and-recovery)
19. [Screens](#19-screens)
20. [Demo script](#20-demo-script)
21. [Project status and known limitations](#21-project-status-and-known-limitations)
22. [Roadmap to a clinical pilot](#22-roadmap-to-a-clinical-pilot)
23. [Team, hackathon rules and submission](#23-team-hackathon-rules-and-submission)
24. [AI tool disclosure and acknowledgements](#24-ai-tool-disclosure-and-acknowledgements)
25. [License](#25-license)

---

## 1. Why this project exists

Oral cancer is a significant public health concern. Early detection is difficult in remote tribal areas because access to specialists is limited. By the time a suspicious lesion is noticed, a patient may be days of travel away from a doctor who can assess it.

**Aarogya Drishti** ("health vision") is a complete, functional, secure digital pathway that connects **patients, health workers, medical officers, specialists, navigators and administrators** in one continuous healthcare workflow: from a village-level oral examination to timely specialist diagnosis and treatment linkage.

It builds on the ideas of oral cancer screening using smartphone images and AI-assisted assessment, inspired by **IISc TANUH - Aarogya Aarohan**.

### The gaps it closes

A villager with a persistent mouth lesion is examined by a frontline worker, but with poor documentation and no photos, referral is delayed, diagnostics are slow, and follow-up is lost.

| Gap | Description |
|---|---|
| Limited access | Specialist services are scarce in remote tribal areas. |
| Weak documentation | Oral abnormalities and screening data are hard to record consistently. |
| Delays | Referrals, diagnostics and follow-up are slow. |
| No coordination | There is no single view of the care journey across roles. |

**Goal: ensure no patient is lost between screening, referral and treatment.** The gap is coordination, not only detection, so this is a workflow platform, not just an image classifier.

### Before and after

| Before: disconnected | After: one connected platform |
|---|---|
| Paper notes, no standard photos | Patient registration + consent, screening + photos |
| Referral by phone, no tracking | AI-assisted review (advisory only), then independent Medical Officer review |
| No view of attendance | Specialist consult + diagnostics with tracked appointments |
| Patient lost to follow-up | Follow-up tasks + district dashboard |

### Vision and objectives

1. **Digital registration:** patient registration and screening documentation.
2. **Image quality + research AI:** quality checks and research-only AI analysis.
3. **Connected clinicians:** health workers linked to Medical Officers and specialists.
4. **Track the journey:** referrals, appointments, investigations and follow-up.
5. **Secure records:** protected records and district aggregate monitoring.
6. **Telugu + low connectivity:** Telugu/English UI and offline workflows.

## 2. What it does

| Capability | Description |
|---|---|
| **Mobile screening app** | A health worker (ASHA/ANM) registers a patient, records consent and exposure history, and captures guided oral photos (cheek, tongue, palate, gums, floor of mouth) on an Android phone. |
| **Image quality check** | OpenCV checks for blur, brightness and glare; poor images trigger a recapture. |
| **Advisory AI** | A pluggable adapter returns a clearly labelled, non-clinical research observation. Clinical workflow continues if the AI service is down. |
| **Doctor and specialist portal** | Role-scoped queue, case viewer with zoom, versioned and signed clinical review, referral creation. |
| **Video consultation** | Time-limited, per-user tokens for a two-party video call. Recording is off. |
| **Appointments** | Video and physical bookings, status history, printable Telugu/English referral slip, missed-visit handling. |
| **Diagnostics tracking** | Investigation states (advised to clinician-reviewed), report upload, clinician-entered diagnosis. |
| **Patient navigation** | Task queue, contact log, transport support, overdue escalation. |
| **Longitudinal record** | One timeline per patient showing every event, actor, time and version. |
| **District dashboard** | Aggregate counts and rates only, with filters and CSV export. No patient identifiers. |
| **Telugu and English** | One-touch language switch on every page. |
| **Offline functionality** | Drafts saved locally and synced later with idempotency, so retries never create duplicates. |
| **Security and privacy** | Patient-level authorization, audit log, private image storage, MFA for privileged roles. |

### The ten application modules

| # | Module | Scope |
|---|---|---|
| A | Registration | Patient record, history, consent |
| B | Oral screening | Guided smartphone photo capture |
| C | AI analysis | Image quality + research-demo output |
| D | Clinical review | Medical Officer and specialist review |
| E | Telemedicine | Secure two-party video |
| F | Appointments | Hospital bookings and referrals |
| G | Diagnostics | Biopsy/pathology tracking |
| H | Patient record | Longitudinal, audited timeline |
| I | Navigation | Follow-up and missed-visit tasks |
| J | District dashboard | Aggregate indicators |

## 3. Non-negotiable safety rules

These are built into the design, not added afterwards.

1. AI output is **advisory** and stored **separately** from the signed clinician conclusion.
2. The app **never confirms or stages cancer** by itself and **never tells a patient they are cancer-free**. Only a clinician-entered pathology result can mark a case as confirmed in the simulation.
3. Every mock or research model output carries the label **NOT FOR CLINICAL USE**, the model version and a timestamp.
4. The clinical workflow must **keep working if the AI service is down**.
5. Every API call is authenticated and authorized at **patient/facility level**, not just by role (deny by default).
6. **No bounding boxes or heatmaps** unless a real supervised localization model exists.
7. **No secrets in Git.** Only `.env.example` with placeholders.
8. Offline records are marked **synced only after server acknowledgement**.

## 4. End-to-end workflow

```
Patient registration & history
        -> Oral screening (photos via mobile app)
        -> AI-assisted assessment (advisory)
        -> Medical Officer review (signed)
        -> Telemedicine consultation
        -> Referral & appointment
        -> Diagnostics (biopsy, reports)
        -> Treatment tracking & follow-up
        -> District dashboard
```

**Data flow:** Health worker PWA -> authenticated FastAPI -> PostgreSQL + private image storage -> AI adapter (research only) -> stored AI observation -> Medical Officer -> video or physical specialist -> diagnostics -> navigator -> aggregate dashboard.

## 5. Users and permissions

| Capability | Health Worker | Medical Officer | Specialist | Navigator | District Admin |
|---|---|---|---|---|---|
| Register patient, consent | Yes (own facility) | No | No | No | No |
| Screening intake, photos | Yes (assigned) | View | View referred only | No | No |
| Run AI analysis | Yes | View | View | No | No |
| Clinical review and sign | No | Yes | Consult notes only | No | No |
| Create referral | No | Yes | Yes (onward) | No | No |
| Book video/physical appointment | No | Yes | Confirm | Yes | No |
| Investigations and pathology result | No | Order/view | Yes | View status only | No |
| Enter final diagnosis | No | Yes | Yes | No | No |
| Navigation tasks | View assigned follow-up | View | No | Yes | No |
| View images | Own cases | Assigned cases | Referred cases | No (unless needed) | No |
| Dashboard | No | Facility summary | No | Own queue | Aggregates only |
| Audit log | No | No | No | No | Aggregate/system view |

## 6. Architecture

```
 ┌────────────────────────── Client applications ──────────────────────────┐
 │ Health Worker PWA │ Medical Officer portal │ Specialist │ Navigator │ Admin │
 └───────────────────────────────┬──────────────────────────────────────────┘
                                 │ HTTPS (REST)
                         ┌───────▼────────┐
                         │  API Gateway   │
                         └───────┬────────┘
   ┌─────────────────────────────▼──────────────────────────────┐
   │ Backend services (FastAPI)                                  │
   │  Auth & RBAC · Patient & Consent · Screening & AI ·         │
   │  Referral & Appointment · Clinical Records · Diagnostics ·  │
   │  Patient Navigation · Notifications · Audit & Monitoring    │
   └───────┬───────────────────────┬─────────────────┬──────────┘
           │                       │                 │
   ┌───────▼────────┐    ┌─────────▼────────┐   ┌────▼─────────────┐
   │ PostgreSQL     │    │ Private object   │   │ AI service       │
   │ (app + audit)  │    │ storage (images, │   │ image QC +       │
   │                │    │ reports)         │   │ mock/research AI │
   └────────────────┘    └──────────────────┘   └──────────────────┘
                                 │
                 External: video consultation (WebRTC SDK), SMS/email gateway
```

Key design decisions:

- **AI is a separate, replaceable service.** The backend talks to it through an adapter contract; if it fails, the case is intact and manual review works.
- **Images never have public URLs.** Clinicians receive short-lived signed URLs (under 5 minutes), and every image view is audited.
- **Authorization is checked in every query**, using role AND patient/facility relationship.
- **Writes are idempotent.** Each retryable write carries an `Idempotency-Key`; offline-created records carry a `client_uuid`.

## 7. Tech stack

| Layer | Choice |
|---|---|
| Frontend | React + TypeScript + Vite PWA, `react-i18next` (Telugu/English) |
| Backend | FastAPI, SQLAlchemy, Alembic, Pydantic |
| Database | PostgreSQL |
| Image storage | Private S3-compatible bucket (MinIO locally) with short-lived signed URLs |
| Image QC | OpenCV (blur, brightness, glare) |
| AI | Pluggable adapter: `mock-v1` by default, optional EfficientNet-B0 research model |
| Video | Approved SDK (for example Jitsi JaaS, Daily, LiveKit) with time-limited tokens |
| Offline | Service worker + IndexedDB queue with idempotency keys |
| Deployment | Docker Compose locally; managed container/VM plus managed Postgres online |
| Monitoring | Structured logs (no PHI), `/healthz`, uptime check |
| Testing | pytest, API integration tests, Playwright mobile E2E |

The stack above is the recommended one. The official problem statement also permits alternatives (React Native or Flutter, Next.js, Node.js, other secure video or cloud services) provided functionality and security are demonstrated. Poster suggestions such as MobileViT-v2 are acceptable alternatives to EfficientNet-B0 for the optional research model.

## 8. Repository structure

```
aarogya-drishti/
├── apps/
│   └── web/            # PWA: health worker, clinician, navigator, admin UI
├── services/
│   ├── api/            # FastAPI, auth, workflow, migrations
│   └── ai/             # quality checker + mock/research adapters
├── infra/              # docker-compose, deployment manifests
├── docs/               # PRD, architecture, ER, API, threat model, test report
├── tests/              # unit, integration, E2E, security
├── .env.example
└── README.md
```

## 9. Getting started

### Prerequisites

- Docker and Docker Compose
- Node.js 20+ and npm (for frontend development)
- Python 3.11+ (for backend development)
- An Android phone with Chrome (for real-device camera testing)

### Run the full stack locally

```bash
git clone <your-repo-url> aarogya-drishti
cd aarogya-drishti
cp .env.example .env          # then fill in placeholder values
docker compose up --build -d  # web, api, ai, postgres, minio
make migrate                  # run Alembic migrations
make seed                     # load synthetic demo data
curl http://localhost:8000/healthz   # expect HTTP 200
```

### Develop each part separately

```bash
# Frontend
cd apps/web
npm install
npm run dev

# Backend
cd services/api
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

### Useful commands

| Command | Purpose |
|---|---|
| `make seed` | Load synthetic demo data |
| `make reset` | Reset demo data to a known state |
| `make test` | Run unit and integration tests |
| `make e2e` | Run Playwright end-to-end tests |

> Adjust the targets above to match your Makefile as you build it.

### Evaluator test accounts

Credentials are provided in the submission package, never in this repository.

| Role | Account |
|---|---|
| Health Worker | _see submission_ |
| Medical Officer | _see submission_ |
| Specialist | _see submission_ |
| Navigator | _see submission_ |
| District Admin | _see submission_ |

**Deployed demo URL:** `https://<your-deployed-url>`

## 10. Configuration

All configuration is through environment variables. Copy `.env.example` to `.env`. **Never commit `.env`.**

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Signing key for session tokens |
| `JWT_EXPIRY_MINUTES` | Short session expiry |
| `S3_ENDPOINT` | Object storage endpoint (MinIO locally) |
| `S3_ACCESS_KEY` / `S3_SECRET_KEY` | Object storage credentials |
| `S3_BUCKET` | Private bucket name |
| `SIGNED_URL_TTL_SECONDS` | Signed URL lifetime (keep under 300) |
| `AI_SERVICE_URL` | Address of the AI service |
| `AI_PROVIDER` | `mock` (default) or `research` |
| `VIDEO_PROVIDER_KEY` / `VIDEO_PROVIDER_SECRET` | Video SDK credentials |
| `VIDEO_TOKEN_TTL_MINUTES` | Video token lifetime (15-30) |
| `MAX_UPLOAD_MB` | Image size limit (for example 8) |
| `CORS_ORIGINS` | Allowed web origins |

## 11. Data model

### Tables

| Table | Purpose | Key rules |
|---|---|---|
| `facilities` | PHC, hospital, diagnostic centre | Mandal/PHC filters |
| `users` | Accounts and roles | Unique login, active flag, MFA for privileged |
| `patients` | Demographics and residence | UUID, duplicate hint, PII restricted |
| `consents` | Purpose-specific consent | Append-only, versioned |
| `screenings` | One screening visit | Clinician findings separate from AI |
| `images` | Oral photo metadata | Private key, checksum, consent gate |
| `ai_analyses` | Model outputs | Always `clinical_use = false` |
| `clinical_reviews` | MO signed review | Versioned, only clinician can finalize |
| `referrals` | Specialist referral | Explicit clinical decision |
| `appointments` | Video and physical visits | Status history, no public links |
| `appointment_status_history` | Status transitions | Append-only |
| `investigations` | Biopsy, pathology, imaging | Separate from AI label |
| `diagnoses` | Clinician-entered result | Only clinician roles |
| `navigation_tasks` | Follow-up work items | Overdue escalation |
| `contact_attempts` | Navigator calls/visits | Linked to task |
| `idempotency_keys` | Safe retries | Unique per user + key |
| `case_state_history` | Case state transitions | Enforced transitions |
| `audit_events` | Who did what | Append-only, no PHI in metadata |

### Relationships

```
facilities 1-N users
facilities 1-N patients (registered_at)
patients 1-N consents | screenings
screenings 1-N images | clinical_reviews
images 1-N ai_analyses
clinical_reviews 1-N referrals
referrals 1-N appointments | investigations | navigation_tasks
investigations 1-N diagnoses
navigation_tasks 1-N contact_attempts
```

`audit_events`, `consents`, `case_state_history` and `appointment_status_history` are **append-only**: the application database role has `UPDATE`/`DELETE` revoked, or a trigger raises an exception.

Full DDL lives in [`docs/`](docs/) and the Alembic migrations in `services/api`.

## 12. Case state machine

```
DRAFT > CONSENTED > SCREENED > AWAITING_CLINICIAN > REVIEWED > REFERRAL_PLANNED
      > APPOINTED > ASSESSED > DIAGNOSTICS_PENDING > RESULT_REVIEWED > CARE_LINKED | CLOSED

Tracked exceptions: RESCHEDULED, MISSED, DECLINED, LOST_CONTACT
```

| Transition | Allowed actor | Guard |
|---|---|---|
| DRAFT -> CONSENTED | Health worker | Imaging consent granted |
| CONSENTED -> SCREENED | Health worker | Screening saved, at least one acceptable image or manual note |
| SCREENED -> AWAITING_CLINICIAN | System | Submit for review |
| AWAITING_CLINICIAN -> REVIEWED | Medical Officer | Review signed |
| REVIEWED -> REFERRAL_PLANNED | Medical Officer | Decision is referral |
| REFERRAL_PLANNED -> APPOINTED | MO / Navigator | Appointment CONFIRMED |
| APPOINTED -> ASSESSED | Specialist | Appointment COMPLETED + outcome |
| ASSESSED -> DIAGNOSTICS_PENDING | Specialist | Investigation ordered |
| DIAGNOSTICS_PENDING -> RESULT_REVIEWED | Clinician | Report received and reviewed |
| RESULT_REVIEWED -> CARE_LINKED / CLOSED | Clinician / Navigator | Diagnosis entered by clinician |

**No AI result may move a case forward by itself.** Any transition not in this table is rejected with **HTTP 409**.

## 13. API overview

All routes require authentication, patient/facility authorization, validation, pagination for lists, standard error codes, and an `Idempotency-Key` header on retryable writes. Interactive OpenAPI docs are served at `/docs`.

| Method + route | Purpose | Who |
|---|---|---|
| `POST /auth/login` | Session/JWT with expiry | All |
| `POST /auth/mfa/verify` | MFA for privileged users | MO, specialist, admin |
| `GET /me` | Current user, role, facility | All |
| `POST /patients` | Register patient (returns duplicate warnings) | Health worker |
| `GET /patients/{id}` | Longitudinal record | Assigned care team |
| `POST /patients/{id}/consents` | Purpose-specific consent | Health worker |
| `POST /screenings` | Create screening + history (accepts `client_uuid`) | Health worker |
| `POST /screenings/{id}/images` | Upload photo (needs consent) | Assigned health worker |
| `GET /images/{id}/url` | Short-lived signed URL, audited | Authorized clinicians |
| `POST /ai/analyze` | Queue non-clinical analysis | Screening user |
| `GET /ai/analyses?image_id=` | AI history | Care team |
| `GET /queue` | Role-scoped work queue | MO, specialist, navigator |
| `POST /reviews` | Save and sign clinical review | Medical Officer |
| `POST /referrals` | Create referral + printable slip | Medical Officer |
| `GET /slots` | Available appointment slots | MO, navigator |
| `POST /appointments` | Book / request | MO, navigator |
| `PATCH /appointments/{id}` | Confirm, reschedule, cancel, attendance, outcome | Clinician / navigator |
| `POST /appointments/{id}/videotoken` | Time-limited token for authorized participant | Participants only |
| `POST /investigations` | Order biopsy/pathology | Specialist, MO |
| `PATCH /investigations/{id}` | Update state, attach report | Specialist |
| `POST /diagnoses` | Clinician-entered result | MO, specialist |
| `PATCH /navigation/{id}` | Contact attempt, status | Navigator |
| `GET /dashboard/summary` | Aggregate counts and rates | District admin |
| `GET /dashboard/export` | Aggregate export only | District admin |
| `GET /healthz` | Health check | Public |

### Standard AI response (research demo)

```json
{
  "analysis_id": "DEMO-AI-009",
  "screening_id": "DEMO-CASE-001",
  "image_quality": "acceptable",
  "provider": "mock",
  "model_version": "mock-v1",
  "research_label": "suspicious_example",
  "score": null,
  "clinical_use": false,
  "requires_clinician_review": true,
  "warning": "NOT FOR CLINICAL USE"
}
```

## 14. Image quality and AI pipeline

### Image checks (`services/ai`)

1. Validate extension and decode with Pillow/OpenCV; reject non-images (for example a renamed PDF).
2. Enforce maximum size (for example 8 MB) and minimum dimensions.
3. Fix EXIF orientation, strip metadata, compute SHA-256.
4. **Blur:** variance of Laplacian below threshold -> recapture.
5. **Brightness:** mean grayscale too low or too high -> recapture.
6. **Glare:** fraction of saturated pixels above threshold -> recapture.
7. Thresholds are calibrated on demo images and recorded in `docs/`.

### AI adapter levels

| Level | What | When |
|---|---|---|
| 1 (required) | Mock adapter with fixed demo labels | Core build |
| 2 (optional) | EfficientNet-B0 fine-tuned on a licensed dataset | Only if the core system is complete |
| 3 (never) | Real clinical use | Not allowed |

Adapter contract: `analyze(image) -> dict`, with timeouts, failure handling, and the model version stored on every row.

If the research model is trained, the repository includes `DATASET_CARD.md` and `MODEL_CARD.md`; source images are split into train/val/test **before** augmentation to avoid leakage; and sensitivity, specificity, precision, F1, confusion matrix and sample sizes are reported. Output labels stay research-only.

### Honest AI claims

- No accuracy, sensitivity or validation claims for the mock adapter, and no heatmap or localization claims.
- OpenCV flags blur, brightness, glare and invalid files; it cannot prove a lesion is absent.
- An optional classifier is shown only with proper dataset licensing and documented limitations (`DATASET_CARD.md`, `MODEL_CARD.md`).
- Misleading diagnostic claims can disqualify a team.

## 15. Telugu and offline-first design

| Area | Implementation |
|---|---|
| i18n | `react-i18next` with `te.json` and `en.json`; no hardcoded English; toggle always visible |
| Fonts | Noto Sans Telugu bundled locally; tested on Android and in PDF |
| Usability | 44px+ touch targets, icons with labels, progress steps, saved confirmation, minimal typing |
| Offline storage | IndexedDB drafts; payloads encrypted with WebCrypto where feasible (browser storage is not fully protected, and this is documented) |
| Sync queue | Each write has a client UUID + `Idempotency-Key`; states: pending, uploading, confirmed, failed |
| Conflicts | Resolution screen; clinician notes are never discarded |

**Offline test:** airplane mode -> create patient and draft -> close and reopen -> reconnect -> sync once -> force retry -> exactly one server record.

## 16. Security and privacy

| Area | Requirement |
|---|---|
| Auth | Argon2/bcrypt, short session expiry, refresh handling, MFA for MO/specialist/admin, no shared accounts |
| Authorization | Role AND patient/facility check in every query; deny by default; tests for each endpoint |
| Transport and storage | HTTPS, encrypted DB and backups, private bucket, signed URLs under 5 minutes |
| Uploads | Decode check, size limit, safe filenames, no path traversal, strip metadata |
| Audit | Image views, exports, clinical edits, diagnosis and permission changes are logged; no PHI in logs |
| Secrets | Environment variables or a secret manager; `.env` is gitignored; rotate after the event |
| Threats covered | Stolen phone, expired/replayed token, IDOR, mass assignment, leaked signed URL, malicious upload, shared video link, backup leak |
| Governance | Synthetic data only; demo reset script; test users disabled after the event; DPDP and telemedicine compliance in the roadmap |

Privacy rules in the UI: no patient names in browser titles, notifications or URLs; a session timeout screen; reminder texts carry no clinical details.

See `docs/threat-model.md` for the full threat model.

## 17. Testing

| Test | Input | Expected |
|---|---|---|
| Normal journey | Consent -> screening -> photo -> review -> video -> biopsy | One continuous record |
| Blurred photo | Defocused image | Recapture warning |
| Invalid file | PDF renamed `.jpg` | Rejected after decode |
| Missing consent | Upload before consent | Rejected with explanation |
| Wrong patient | DoctorA opens unrelated case | 403 |
| AI down | Stop AI container | Case intact; manual review works |
| Network loss | Disconnect during upload | Retry, no duplicate image |
| Double submit | Same POST + same key | Single record |
| False-negative AI | AI low, clinician suspicious | Referral still possible |
| False-positive AI | AI suspicious, clinician disagrees | Discrepancy documented |
| Missed appointment | Mark missed | Navigation task created |
| Bad video token | Expired or wrong token | Join denied; reschedule possible |
| Long Telugu text | Switch language, PDF, mobile | No overflow |
| Concurrent edit | Two users edit one review | Conflict or version error |
| Dashboard | 12 cases, 4 referrals seeded | Exact counts |
| Audit | Change pathology status | Actor/time/action stored |
| Invalid transition | SCREENED -> CARE_LINKED | 409 |
| Oversized image | Over limit | 413 |
| Traversal filename | `../../x.jpg` | Sanitized or rejected |

Evidence produced: pytest unit tests, API integration tests, Playwright mobile E2E, authorization tests, offline demo recording, and a known-issues list. Security tests are written first, then fixed.

### Mandatory tests (from the hackathon brief)

- Telugu language switch works
- Offline registration sync works
- Blurry/poor image handling
- Unauthorized access blocked
- Doctor can override the AI result
- Missed appointment follow-up flow
- Network-failure recovery
- Audit trail visible

Judges may run surprise functional and security tests, so test failure scenarios, not just happy paths.

### Evidence policy

A test is marked **Passed** only after it has been executed, with command output, a screenshot, a recording or a database count as evidence. No evidence means no pass. Planned evidence per scenario: end-to-end journey (video + records), recapture (screenshot), missing consent (API test output), unauthorized access (test log), AI failure (stopped-container demo), network loss (airplane-mode video), duplicate submission (DB count), Telugu switching (phone screenshots), doctor overrides AI (review record), missed appointment (task screenshot), invalid video link (call log), dashboard accuracy (seed vs dashboard), audit trail (audit export).

## 18. Deployment and recovery

Free hosting tiers may sleep or wipe files, so a persistent database and object storage are used, with a local Docker route as backup.

1. `docker compose up --build -d` (web, api, ai, postgres, minio)
2. Run Alembic migrations, then `make seed`
3. Verify `GET /healthz` returns 200
4. Open the HTTPS URL on Android; test camera, reload, offline
5. Restart the API; confirm data persists
6. Confirm the bucket is private and signed URLs expire
7. Run automated tests against the deployed URL
8. Take a DB backup and restore into a fresh environment
9. `make reset` restores demo data
10. After the event: disable test users, rotate secrets

## 19. Screens

Every screen has an ID. Health worker screens (HW) are mobile-first; Medical Officer, specialist, navigator and admin screens (MO, SP, NV, AD) target laptop/tablet but must work on a phone.

**Health worker (10 screens):** HW-01 Login -> HW-02 Home -> HW-03 Register -> HW-04 Consent -> HW-05 History -> HW-06 Camera -> HW-07 Photo check (retake loop) -> HW-08 AI result -> HW-09 Submit -> HW-10 Sync -> MO-01 Queue

**Clinical (12 screens):** MO-01 Queue -> MO-02 Case view -> MO-03 Review (sign) -> decision (recapture / routine / refer) -> MO-04 Referral -> MO-05 Book visit -> SP-01 Queue -> SP-02 Consult -> SP-03 Order test -> SP-04 Report -> SP-05 Diagnosis (sign) -> NV-01 Follow-up. TL-01 Timeline is available from any case.

**Navigator and admin (5 screens):** NV-01 Task queue -> NV-02 Task detail -> NV-03 Treatment (or MO-05 reschedule if missed); AD-01 Dashboard -> AD-02 Export.

### Design rules

- Sticky bottom bar with one full-width **Next** button, at least 48px tall; body text 16px or larger.
- Every string comes from the te/en dictionary; the language toggle is visible on every page.
- Status colours: grey draft, blue pending, green confirmed/signed, amber warning or AI advisory, red failed/urgent. **A patient is never coloured as "cancer" or "healthy".**
- AI card: always amber, with **NOT FOR CLINICAL USE**, model version and timestamp; no score, heatmap or box.
- Clinician-signed items show a green badge with doctor name, time and version.
- Every write shows a visible "Saved" confirmation; failures show a clear retry; an offline banner appears when disconnected.

## 20. Demo script (10 minutes)

1. Open the deployed app in Telugu on a phone.
2. Register a synthetic patient: 46-year-old from G. Madugula, tobacco chewing, persistent white cheek patch.
3. Record consent and symptoms.
4. Upload a good demo image, then a blurry one (recapture shown).
5. Run the mock AI; stop the AI service and show graceful recovery.
6. Log in as Medical Officer, review, disagree with the AI if relevant, sign the decision.
7. Book a video consultation and join with a second account.
8. Create a physical referral, mark attendance, upload a simulated pathology report.
9. Specialist enters the final simulated diagnosis.
10. Navigator task updated; dashboard counts correct.
11. Show unauthorized access blocked, the audit trail, and offline sync.
12. Explain limitations and the roadmap to an approved clinical pilot.

## 21. Project status and known limitations

Each capability moves through **PLANNED -> IMPLEMENTED -> VERIFIED (with evidence)**. Update this table only when you have evidence.

| Area | Status | Evidence |
|---|---|---|
| A. Registration and history | Planned | - |
| B. Smartphone screening | Planned | - |
| C. AI assessment (mock adapter) | Planned | - |
| D. MO and specialist review | Planned | - |
| E. Video consultation | Planned | - |
| F. Physical appointments | Planned | - |
| G. Diagnostics | Planned | - |
| H. Longitudinal record | Planned | - |
| I. Navigation | Planned | - |
| J. District dashboard | Planned | - |
| Telugu UI | Planned | - |
| Offline sync | Planned (verify on a real Android device before claiming) | - |

Executed / passed tests: none yet.

### Known limitations

- Synthetic data only; not validated on real patients.
- The default AI is a mock adapter returning fixed labels. Any research model is unvalidated and non-clinical.
- Browser-side offline storage is not fully protected.
- Not certified under DPDP, telemedicine guidelines or medical-device regulation.
- Free-tier hosting may be slow or sleep.

_Update this section with the real known-issues list as the build progresses._

## 22. Roadmap to a clinical pilot

1. Clinical validation of any AI model on a representative, ethically approved dataset.
2. Regulatory review (medical-device classification, telemedicine practice guidelines).
3. DPDP Act compliance: consent management, data localisation, retention, breach process.
4. Integration with national and state health systems (for example ABDM) and existing PHC registries.
5. Production-grade security review and penetration test.
6. Field pilot with trained ASHA/ANM workers and supervising clinicians, with outcome tracking.
7. Expanded language support, voice guidance and low-bandwidth optimisation.

## 23. Team, hackathon rules and submission

| Suggested role | Core responsibilities | Definition of done | Member |
|---|---|---|---|
| Backend + database | FastAPI, schema, APIs, integration | Secure end-to-end APIs | _name_ |
| Frontend / mobile | PWA, camera, offline, bilingual forms | Screening works on a real Android phone | _name_ |
| AI integration | OpenCV quality, mock adapter, optional research model | Reproducible output, no medical claims | _name_ |
| Clinical workflow | Doctor portal, referral, video, navigator, dashboard | Complete simulated journey | _name_ |
| QA / security / DevOps | Tests, authorization, deployment, monitoring | Negative tests, secure deploy, runbook | _name_ |

Teams of fewer than five combine roles but keep every row covered. Agree one interface contract early and integrate twice a day.

Team: _name_ · AAROHAN centre: _name_

Working agreements: 15-minute daily stand-up, merge twice a day, a daily development log with commits, vertical slices with passing tests before moving on. Contribution record is kept in the Git history and `docs/dev-log.md`.

**Code freeze:** 4 Nov 2026, 10:00 AM IST.

### Hackathon rules

- Team size: minimum 2, maximum 5 students; only currently enrolled AAROHAN students; each student is on only one team; open to all AAROHAN centres.
- Open-book: AI coding tools and open-source tools are allowed with proper disclosure.
- Synthetic or authorized demonstration data only. **Real patient data is prohibited.**
- Disqualifiers: real patient data, presenting AI as clinical, exposed records, plagiarism, hardcoded secrets.
- Prize: Rs 10,000 for the winning team.

### Schedule

| Date | Focus |
|---|---|
| 27 Oct | Launch and team registration |
| 28 Oct | Clinical workflow study and planning (UI/UX, schema, setup) |
| 29 Oct | Registration, consent and screening forms; mobile module |
| 30 Oct | Image capture and AI integration |
| 31 Oct | Doctor portal and review; **minimum complete system** |
| 1 Nov | Video consultation and appointments |
| 2 Nov | Diagnostics, tracking, Telugu and security |
| 3 Nov | Testing, deployment, semifinal review, rehearsal |
| 4 Nov | Code freeze 10:00 AM IST, final demonstration and awards |

### Evaluation (100 marks)

| Criterion | Marks |
|---|---|
| Complete workflow and integration | 20 |
| AI engineering and safe integration | 15 |
| Medical review, teleconsultation and appointments | 15 |
| Records, diagnostics and patient navigation | 15 |
| Security, privacy and data governance | 15 |
| Telugu usability and offline capability | 10 |
| Deployment, code quality and documentation | 10 |

Tie-breakers: patient workflow, then security/privacy, then AI safety/integration. A safety violation can disqualify a team regardless of technical score.

### Submission checklist

- [ ] Source code repository with daily commits and contribution record
- [ ] Deployed HTTPS demo URL and evaluator test accounts
- [ ] README (install, architecture, stack, credentials, known limits)
- [ ] Architecture diagram with trust boundaries; ER diagram; role-access matrix
- [ ] API and database documentation (OpenAPI with examples); environment variable table (no secrets)
- [ ] AI model explanation: model card, dataset card, validation and limitations
- [ ] Security note and threat model
- [ ] Test report with failure cases and offline tests
- [ ] Deployment, backup and rollback guide
- [ ] Synthetic dataset and reset command
- [ ] Telugu UI demonstration
- [ ] 5-minute product demonstration video
- [ ] Disclosure of AI tools, templates and third-party code
- [ ] Roadmap from demo to approved clinical pilot
- [ ] Live presentation on 4 Nov 2026

## 24. AI tool disclosure and acknowledgements

- **AI tools used:** _list assistants and how they were used (for example design drafts, code suggestions). All generated code was inspected and tested before committing._
- **Templates and third-party code:** _list libraries, templates and licensed images/datasets with licenses._
- **Inspiration:** IISc TANUH - Aarogya Aarohan oral cancer screening.
- **Organised by:** ITDA Paderu, Alluri Sitharama Raju District, Andhra Pradesh, with AAROHAN (AI & Software Skills for a Brighter Tomorrow).

## 25. License

_Choose a license (for example MIT or Apache-2.0) and add a `LICENSE` file._

---

*From a village-level oral examination to timely specialist diagnosis and treatment: building the complete digital pathway.*
