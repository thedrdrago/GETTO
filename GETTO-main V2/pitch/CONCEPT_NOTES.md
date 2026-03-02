# JETTO — Core Concept Notes
_Vlad × Timour session, March 2026_

---

## One-line idea
A single source of truth for construction drawings — always shows the latest version, enables on-drawing comments, and notifies the right people instantly.

---

## The core problem this solves
Architects, contractors, and subcontractors communicate via WhatsApp. Drawings get buried, old versions get built from, and there is no audit trail. JETTO replaces the chaos with one organised vault per project.

> Applies to any "discombobulated construction market" — Sicily, UK, Ukraine, anywhere.

---

## Who uses it

| Role | What they do |
|---|---|
| **Architect** | Organises the folder structure, uploads drawings, creates new versions, resolves comments |
| **Contractor** | Views current drawings, leaves comments on the drawing itself, gets upload notifications |
| **Subcontractor** | Same as Contractor but only has access to folders relevant to their scope of work (e.g. cabinetry, landscaping) |
| **Client / Owner** | The person who is pitched to and pays — creates the accountability that pushes adoption |

---

## The workflow loop

1. Architect uploads a drawing → it lands in the right folder
2. Contractor + relevant subcontractors are notified
3. Contractor opens the drawing, pins a comment directly on it
4. Architect is notified → revises → uploads a new version
5. Old version is automatically archived (legal requirement; all versions kept forever)
6. Repeat until drawing is marked **Executed**

---

## The Vault (folder structure)

- Customisable hierarchy, defined by the architect per project
- No fixed template — architect chooses the organisation logic (by room, by trade, by drawing type, etc.)
- Example depth: **Project → Piano Terra → Cucina → Montature Mobili → drawing**
- A kitchen can have 50+ drawings — sub-folders keep them navigable
- View always shows the **current version**; archived versions accessible separately

### Future vision (aspirational)
Upload the master plan as the root. Tapping a room on the plan navigates directly to that room's folder. Harder to build but the most intuitive UX.

---

## Drawing detail view

- **Comments live on the drawing itself** — pin a dot, type a comment, thread reply
- Maximise the drawing canvas; metadata (version, status, assignee) sits below
- Architect side: can upload a new version
- Builder side: can only comment — intentionally simple

---

## Notifications

- New drawing uploaded → notify the contractor + all subcontractors with access to that folder
- Comment added → notify the architect
- Channel: **WhatsApp** (the market is Italian/Sicilian; Telegram is not relevant here)
- Targeted — only relevant parties get pinged, not a broadcast to everyone

---

## Why not Dropbox / Google Drive?

1. **Comments on the drawing** — Dropbox does not support pinned, threaded markup on PDFs
2. **Live targeted notifications** — Dropbox sends daily email digests that nobody reads; JETTO sends a WhatsApp message to the right person the moment something changes

---

## Go-to-market angle

**Sell to the client (building owner), not the architect.**

- Architects resist adding new software to their workflow
- The client cares about cost overruns and miscommunication — JETTO is a direct answer to both
- Once the client mandates it, the architect and contractor adopt it

Architect benefit (once inside): fewer redraws caused by miscommunication → hours saved

---

## Dashboard — what matters

- Active projects
- Total drawings
- Open comments

Remove "completion %" — not meaningful enough.

---

## Subcontractor access model

Invited into specific **folders** only, like adding a collaborator to a GitHub repo.
- Subcontractor for cabinets → sees only the cabinetry folders
- Gets notified when a comment is left in their scope, or when a new drawing for their scope is uploaded
- Contractor can also allow a technical person (e.g. site manager) to run their account or be added as a collaborator

---

## Open questions / to figure out

- Onboarding flow for the architect (folder setup UX — needs to be quick but customisable)
- Folder structure templates vs fully bespoke (offer templates as a starting point)
- Whether the visual master-plan tap-to-navigate is V1 or later
- Exact WhatsApp integration mechanism (Business API vs link-based)
