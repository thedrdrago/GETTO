# JETTO MVP Technical Specification

**Version:** 1.0  
**Date:** January 2026  
**Purpose:** Actionable technical requirements for building JETTO MVP (Months 0–6)

---

## 1. EXECUTIVE SUMMARY

JETTO MVP is a **mobile-first drawing vault** for Sicilian construction execution. Core functionality: upload, version control, comment, notify, role-based access. Target: 18 live pilot projects by Month 6.

**Tech Stack (recommended):**
- **Frontend:** React Native (iOS/Android) or Progressive Web App (PWA) with Next.js
- **Backend:** Supabase (PostgreSQL + Auth + Storage + Realtime) or Firebase
- **Storage:** Cloud object storage (Supabase Storage / AWS S3)
- **PDF Rendering:** PDF.js or PSPDFKit-like library (mobile-optimized)
- **Notifications:** Email (SendGrid/Resend) + optional WhatsApp Business API integration

**Non-Goals for MVP:**
- Desktop-first UI
- BIM/3D collaboration
- In-app chat (use WhatsApp)
- Advanced analytics dashboard (post-MVP)
- Payment processing (manual invoicing for pilots)

---

## 2. USER ROLES & PERMISSIONS

| **Role** | **Permissions** | **Key Actions** | **Free/Premium** |
|---|---|---|---|
| **Architect/Engineer** | Upload, Revise, Delete (own drawings), Comment | Create project, upload drawings, revise versions, respond to comments | Premium (€10–20/mo) after 1 project |
| **PM/Head of Works** | Upload, Assign, Mark Ready/Executed, Comment, View All | Enforce drawing status, assign to builders, oversee execution | Premium (€10–20/mo) |
| **Builder/Subcontractor** | View assigned + all drawings, Comment, Mark Executed (assigned only) | Execute drawings, comment issues, search vault | Free forever |
| **Owner (Client)** | View all drawings, Client Comments (visually distinct), No status changes | Monitor progress, leave feedback (non-blocking) | Free (1 project) or €10/mo (unlimited) |

**Permission Matrix:**

| **Action** | **Architect** | **PM** | **Builder** | **Owner** |
|---|---|---|---|---|
| Create Project | ✅ | ✅ | ❌ | ❌ |
| Upload Drawing | ✅ | ✅ | ❌ | ❌ |
| Revise Version | ✅ (own) | ✅ (any) | ❌ | ❌ |
| Delete Drawing | ✅ (own, pre-execution) | ✅ (any, pre-execution) | ❌ | ❌ |
| Assign Drawing | ✅ | ✅ | ❌ | ❌ |
| Comment (standard) | ✅ | ✅ | ✅ | ❌ |
| Client Comment | ❌ | ❌ | ❌ | ✅ |
| Mark Ready | ❌ | ✅ | ❌ | ❌ |
| Mark Executed | ❌ | ✅ | ✅ (assigned only) | ❌ |
| View Audit Trail | ✅ | ✅ | ✅ (assigned) | ✅ |
| Search All Drawings | ✅ | ✅ | ✅ | ✅ |
| Export Archive | ✅ (Premium) | ✅ (Premium) | ❌ | ✅ (Premium) |

---

## 3. CORE ENTITIES & DATA MODEL

### 3.1 Database Schema (PostgreSQL / Firestore)

**Table: `projects`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `name` | VARCHAR(255) | Project name |
| `location` | VARCHAR(255) | Physical location (e.g., "Palermo, Sicily") |
| `phase` | ENUM | "planning" or "execution" |
| `owner_id` | UUID | User ID of project creator |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last update |
| `is_archived` | BOOLEAN | Soft delete flag |

**Table: `project_collaborators`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `project_id` | UUID | Foreign key → projects |
| `user_id` | UUID | Foreign key → users |
| `role` | ENUM | "architect", "pm", "builder", "owner" |
| `invited_at` | TIMESTAMP | Invitation timestamp |
| `accepted_at` | TIMESTAMP | Acceptance timestamp (null if pending) |

**Table: `drawings`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `project_id` | UUID | Foreign key → projects |
| `folder_id` | UUID | Foreign key → folders (nullable – drawings at root if NULL) |
| `title` | VARCHAR(255) | Drawing title (e.g., "Ground Floor Plan A") |
| `file_type` | ENUM | "pdf", "dwg", "jpg", "png" |
| `storage_url` | TEXT | Cloud storage path |
| `current_version_id` | UUID | Foreign key → drawing_versions (latest) |
| `status` | ENUM | "draft", "ready", "executed", "archived" |
| `uploaded_by` | UUID | Foreign key → users |
| `assigned_to` | UUID | Foreign key → users (builder/subcontractor) |
| `created_at` | TIMESTAMP | Upload timestamp |
| `updated_at` | TIMESTAMP | Last update |

**Table: `drawing_versions`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `drawing_id` | UUID | Foreign key → drawings |
| `version_number` | INT | Incremental version (1, 2, 3…) |
| `storage_url` | TEXT | Cloud storage path for this version |
| `uploaded_by` | UUID | Foreign key → users |
| `upload_timestamp` | TIMESTAMP | Version creation time |
| `change_notes` | TEXT | Optional revision description |
| `is_latest` | BOOLEAN | True for current version only |

**Table: `comments`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `drawing_id` | UUID | Foreign key → drawings |
| `version_id` | UUID | Foreign key → drawing_versions (anchored to specific version) |
| `user_id` | UUID | Foreign key → users |
| `comment_type` | ENUM | "standard", "client_comment" |
| `text` | TEXT | Comment body |
| `annotation_data` | JSONB | PDF coordinates (x, y, page) for visual anchor |
| `is_resolved` | BOOLEAN | Resolved status |
| `created_at` | TIMESTAMP | Comment timestamp |
| `resolved_at` | TIMESTAMP | Resolution timestamp |

**Table: `notifications`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `user_id` | UUID | Foreign key → users (recipient) |
| `project_id` | UUID | Foreign key → projects |
| `drawing_id` | UUID | Foreign key → drawings (nullable) |
| `type` | ENUM | "new_drawing", "new_version", "comment_added", "status_change", "assignment" |
| `message` | TEXT | Notification message |
| `is_read` | BOOLEAN | Read status |
| `created_at` | TIMESTAMP | Notification timestamp |

**Table: `folders` (Hierarchical vault structure – replaces flat `areas`)**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `project_id` | UUID | Foreign key → projects |
| `parent_id` | UUID | Foreign key → folders (NULL = root-level folder) |
| `name` | VARCHAR(255) | Folder name (e.g., "Piano Terra", "Cucina") |
| `path` | TEXT | Materialized path: `/{project_id}/{folder_id}/` or `/{project_id}/{parent_id}/{folder_id}/` |
| `position` | INT | Sort order among siblings (for manual reordering) |
| `created_by` | UUID | Foreign key → users |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last update |

> **Pattern:** Adjacency List + Materialized Path hybrid. `parent_id` handles tree navigation; `path` enables fast subtree queries via `LIKE '/proj/folder/%'` and instant breadcrumb rendering. See `spec/VAULT_FOLDER_STRUCTURE.md` for full technical guide. Max depth: 4 levels.

**Table: `users`**
| Field | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `email` | VARCHAR(255) | User email (unique) |
| `phone` | VARCHAR(50) | Optional phone for WhatsApp |
| `display_name` | VARCHAR(255) | Full name |
| `language` | ENUM | "it", "en" (default: "it") |
| `subscription_tier` | ENUM | "free", "premium" |
| `created_at` | TIMESTAMP | Account creation |

---

### 3.2 Key Relationships

```
projects (1) ──→ (many) project_collaborators
projects (1) ──→ (many) folders
projects (1) ──→ (many) drawings
folders  (1) ──→ (many) folders    (self-referential: parent → children)
folders  (1) ──→ (many) drawings   (folder_id on drawings)
drawings (1) ──→ (many) drawing_versions
drawings (1) ──→ (many) comments
drawing_versions (1) ──→ (many) comments (anchored to version)
users (1) ──→ (many) notifications
```

---

## 4. USER STORIES (MVP)

### 4.1 Architect/Engineer Stories

**Story 1: Create Project & Invite Team**
- **As an** Architect
- **I want to** create a new project and invite collaborators via WhatsApp/email
- **So that** my team has a central drawing vault from Day 1

**Acceptance Criteria:**
- Can set project name, location, phase (planning/execution)
- Can generate WhatsApp invite link (deep link to project signup)
- Can send email invites with project context
- Invitees land in onboarding flow (role selection, minimal tutorial)

---

**Story 2: Upload Drawing**
- **As an** Architect
- **I want to** upload a drawing (PDF/DWG/image) with title and optional area
- **So that** the team has access to the latest design

**Acceptance Criteria:**
- Upload via mobile (camera/file picker) or web
- Set title, area, assign to specific builder (optional)
- Default status: "Draft"
- All collaborators notified (in-app + email)
- File uploaded to cloud storage; URL stored in DB

---

**Story 3: Revise Drawing (New Version)**
- **As an** Architect
- **I want to** upload a new version of an existing drawing
- **So that** old versions are archived and the latest is clear

**Acceptance Criteria:**
- Select existing drawing → "Upload New Version"
- Old version marked `is_latest = false`, archived
- New version marked `is_latest = true`, version number incremented
- Notification sent: "Drawing [title] updated to v[N]"
- Old version accessible via "See older versions" (hidden by default in search/list)

---

### 4.2 PM/Head of Works Stories

**Story 4: Assign Drawing to Builder**
- **As a** PM
- **I want to** assign a specific drawing to a builder/subcontractor
- **So that** they know which drawings they are responsible for executing

**Acceptance Criteria:**
- Select drawing → "Assign to [Builder Name]"
- Builder sees assigned drawings at top of their dashboard
- Notification sent: "You've been assigned [drawing title]"

---

**Story 5: Mark Drawing "Ready for Execution"**
- **As a** PM
- **I want to** review a drawing and mark it "Ready for Execution"
- **So that** builders know it's approved and can be executed on-site

**Acceptance Criteria:**
- Drawing status changes from "Draft" → "Ready"
- Assigned builder(s) notified
- "Ready for Execution" drawings appear at top of builder dashboard

---

### 4.3 Builder/Subcontractor Stories

**Story 6: View Assigned Drawings**
- **As a** Builder
- **I want to** see which drawings I'm assigned to execute
- **So that** I can prioritize my on-site work

**Acceptance Criteria:**
- Dashboard shows "Assigned to You" section at top
- Drawings sorted by status (Ready → Draft)
- Can search all project drawings, but assigned ones prioritized

---

**Story 7: Comment on Drawing (Mark Issue)**
- **As a** Builder
- **I want to** mark a specific location on a PDF drawing and leave a comment
- **So that** the architect knows exactly what needs clarification or revision

**Acceptance Criteria:**
- Tap drawing → "Add Comment"
- Tap location on PDF → comment box appears
- Comment anchored to (x, y, page, version)
- Architect notified immediately
- Comment visible to all collaborators

---

**Story 8: Mark Drawing "Executed"**
- **As a** Builder
- **I want to** mark a drawing as "Executed" when I've completed the work on-site
- **So that** the team knows this task is complete and traceable

**Acceptance Criteria:**
- "Mark Executed" button available only for assigned drawings in "Ready" status
- Status changes to "Executed"
- Timestamp logged
- PM and architect notified

---

### 4.4 Owner Stories

**Story 9: View Project Progress**
- **As an** Owner
- **I want to** view all drawings and their execution status
- **So that** I can monitor progress without interfering with execution

**Acceptance Criteria:**
- Read-only access to all drawings
- Can view comments, audit trail, versions
- Cannot change status or assign drawings
- Dashboard shows % of drawings executed

---

**Story 10: Leave Client Comment**
- **As an** Owner
- **I want to** leave feedback on a drawing without blocking execution
- **So that** my preferences are documented but don't disrupt the workflow

**Acceptance Criteria:**
- "Client Comment" button (visually distinct: different color/icon)
- Comments labeled "Client Feedback" in UI
- Does not trigger "revision required" workflow (informational only)

---

### 4.5 Cross-Role Stories

**Story 11: Search Drawings**
- **As any** User
- **I want to** search drawings by title, area, or status
- **So that** I can quickly find what I need on-site or in the office

**Acceptance Criteria:**
- Search bar always accessible
- Filters: status, area, assigned to me, file type
- Results show only latest version by default
- "Include older versions" checkbox (off by default)

---

**Story 12: View Audit Trail**
- **As any** User
- **I want to** see full history of actions on a drawing (uploads, comments, status changes)
- **So that** I can trace decisions and resolve disputes

**Acceptance Criteria:**
- "History" tab on drawing detail page
- Chronological log: timestamp, user, action, notes
- Exportable (Premium feature)

---

## 5. NONFUNCTIONAL REQUIREMENTS

### 5.1 Performance
- **PDF Rendering:** Sub-2-second load time for typical construction drawing (2–10 MB PDF) on 4G mobile connection
- **Comment Anchoring:** Tap-to-comment must feel instant (<200ms latency)
- **Search:** Results appear within 500ms for 100-drawing vault
- **Concurrent Users:** Support 20 simultaneous users per project without lag

### 5.2 Mobile-First Design
- **Primary Platform:** iOS + Android (React Native or PWA)
- **Offline Mode (Post-MVP):** Cache last 10 viewed drawings for offline viewing
- **Touch Targets:** Minimum 44×44px for all interactive elements
- **Responsive:** Works on screens 320px–428px wide (iPhone SE → iPhone 14 Pro Max)

### 5.3 Reliability & Uptime
- **Uptime Target:** 99.5% (acceptable for MVP; upgrade to 99.9% post-launch)
- **Automated Backups:** Daily PostgreSQL backups; retention 30 days
- **Disaster Recovery:** Restore from backup within 4 hours

### 5.4 Security & GDPR
- **Authentication:** Email/password + optional Google OAuth; phone/SMS for WhatsApp integration
- **GDPR Compliance:** EU-based hosting (Hetzner, Scaleway) or GDPR-certified cloud (AWS eu-south-1)
- **Data Encryption:** TLS 1.3 in transit; AES-256 at rest (Supabase default)
- **User Data Deletion:** Account deletion removes all personal data within 30 days; project data anonymized
- **Audit Logs:** All actions logged (user, timestamp, IP) for 12 months

### 5.5 Scalability (Post-MVP)
- **Storage:** Start with 50GB total (pilot phase); scale to 500GB+ (Year 1)
- **Database:** PostgreSQL supports 10k+ projects without sharding
- **CDN:** Use CloudFlare or Supabase CDN for drawing delivery (reduce latency)

### 5.6 Localization
- **Language:** Italian (default), English (secondary)
- **Date/Time:** European format (DD/MM/YYYY, 24h clock)
- **Notifications:** Translated templates (Italian for Sicily pilots)

---

## 6. MVP FEATURE SCOPE CHECKLIST

### ✅ Must-Have (MVP Blocker)
- [ ] User authentication (email/password)
- [ ] Create project + invite collaborators
- [ ] Upload drawing (PDF, DWG, JPG, PNG)
- [ ] Version control (new upload archives old)
- [ ] Comment anchored to PDF location + version
- [ ] Role-based permissions (architect, PM, builder, owner)
- [ ] Drawing status lifecycle (draft → ready → executed)
- [ ] Notifications (in-app + email): new upload, comment, status change
- [ ] Search (by title, status; latest version only by default)
- [ ] Mobile-optimized PDF viewer
- [ ] Audit trail (view-only in MVP)

### 🟡 Should-Have (Launch Early If Possible)
- [ ] WhatsApp invite link generation (deep link)
- [ ] **Hierarchical folder structure** (Dropbox-style; architect customizable per project — see `spec/VAULT_FOLDER_STRUCTURE.md`)
- [ ] "Assigned to Me" dashboard filter
- [ ] Optional WhatsApp notification ping ("You have a new notification in JETTO")
- [ ] Export drawing + comments as PDF (Premium feature)

### ❌ Won't-Have (Post-MVP)
- Desktop-native app (PWA is sufficient)
- In-app chat (use WhatsApp)
- Advanced analytics dashboard
- Automated payment processing (manual invoicing for pilots)
- BIM/3D viewer
- Offline mode
- Multi-language (beyond IT/EN)

---

## 7. IMPOSSIBLE STATES (EDGE CASES TO PREVENT)

| **Impossible State** | **Prevention Logic** |
|---|---|
| **Executing an old drawing** | "Mark Executed" only available for latest version; old versions locked to "Archived" status |
| **Ambiguous "latest" version** | DB constraint: only one `drawing_version` per `drawing_id` can have `is_latest = true` |
| **Comments detached from version** | Comments reference both `drawing_id` and `version_id`; version deletion cascades safely |
| **Builder assigns drawing** | Permissions matrix enforced at API level; UI hides assign button for builder role |
| **Owner changes status** | DB-level role check; owner writes limited to `client_comments` table |
| **Deleting drawing with executed status** | Soft delete only; executed drawings cannot be hard-deleted (audit trail requirement) |
| **Multiple "current versions" after revision** | Transaction: (1) Set old `is_latest = false`, (2) Create new version `is_latest = true`, (3) Update `drawings.current_version_id` |

---

## 8. API ENDPOINTS (High-Level)

### Projects
- `POST /api/projects` – Create project
- `GET /api/projects/{id}` – Get project details
- `POST /api/projects/{id}/invite` – Invite collaborator (email/WhatsApp)

### Drawings
- `POST /api/drawings` – Upload drawing
- `POST /api/drawings/{id}/versions` – Upload new version
- `GET /api/drawings/{id}` – Get drawing + current version
- `GET /api/drawings/{id}/versions` – List all versions
- `PATCH /api/drawings/{id}/status` – Update status (draft/ready/executed)
- `PATCH /api/drawings/{id}/assign` – Assign to user

### Comments
- `POST /api/comments` – Create comment
- `GET /api/drawings/{id}/comments` – List comments for drawing
- `PATCH /api/comments/{id}/resolve` – Mark comment resolved

### Notifications
- `GET /api/notifications` – List user notifications
- `PATCH /api/notifications/{id}/read` – Mark as read

### Folders
- `GET /api/projects/{id}/folders` – List root folders for project
- `GET /api/folders/{id}/children` – List direct children of a folder
- `GET /api/folders/{id}` – Get single folder (includes path for breadcrumb)
- `POST /api/folders` – Create new folder
- `PATCH /api/folders/{id}` – Rename or reorder folder
- `PATCH /api/folders/{id}/move` – Move folder to a new parent
- `DELETE /api/folders/{id}` – Delete folder

### Search
- `GET /api/search?q={query}&status={status}&folder={folder_id}` – Search drawings within folder subtree

---

## 9. TECH STACK RECOMMENDATION

**Option A: Supabase (Recommended for MVP)**
- **Backend:** PostgreSQL + Supabase Auth + Realtime subscriptions
- **Storage:** Supabase Storage (S3-compatible)
- **Pros:** Fast setup, built-in auth, real-time notifications, GDPR-compliant EU hosting
- **Cons:** Vendor lock-in (mitigated by PostgreSQL portability)

**Option B: Firebase**
- **Backend:** Firestore + Firebase Auth + Cloud Functions
- **Storage:** Firebase Storage
- **Pros:** Excellent mobile SDKs, fast prototyping
- **Cons:** NoSQL (less ideal for relational data like versions), US-based (GDPR requires EU region selection)

**Option C: Custom (Overkill for MVP)**
- **Backend:** Node.js (Express/Fastify) + PostgreSQL
- **Storage:** AWS S3 + CloudFront CDN
- **Pros:** Full control
- **Cons:** Slower to build, higher infra management burden

**MVP Recommendation:** **Supabase** for speed + GDPR + real-time capabilities.

---

## 10. DELIVERY MILESTONES

| **Week** | **Milestone** | **Deliverable** |
|---|---|---|
| **1–2** | Setup + Core Schema | Supabase project, DB schema deployed, auth working |
| **3–4** | Upload + Version Control | Drawing upload functional; version logic working |
| **5–6** | Comments + Notifications | PDF commenting anchored; email notifications live |
| **7** | Role Permissions + Status | Role matrix enforced; draft/ready/executed flow complete |
| **8** | Mobile UI Polish | Mobile-optimized; PDF rendering sub-2sec; pilot-ready |

**Pilot Launch:** End of Week 8 → 6 architects × 3 projects = 18 live projects.

---

## 11. ACCEPTANCE CRITERIA FOR MVP "DONE"

MVP is considered **shippable to pilots** when:
- [ ] 6-step loop functional (Upload → Comment → Revise → Ready → Execute → Archive)
- [ ] All 4 user roles working with correct permissions
- [ ] PDF rendering works on iOS Safari + Android Chrome
- [ ] Notifications sent via email for key actions (upload, comment, status change)
- [ ] Search returns correct results (latest version only by default)
- [ ] Audit trail visible (even if not exportable yet)
- [ ] Mobile UX tested on 3+ devices (iPhone, Android, tablet)
- [ ] GDPR-compliant (data deletion implemented, EU hosting confirmed)
- [ ] At least 2 architects have used it for 1 real project without critical bugs

---

**END OF MVP SPEC**  
*This document is actionable for hiring a contract developer. Share with dev candidates to assess technical fit.*
