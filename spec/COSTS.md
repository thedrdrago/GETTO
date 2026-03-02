# JETTO Bootstrap Budget & Build Plan

**Version:** 1.0  
**Date:** January 2026  
**Purpose:** Define realistic cost scenarios, what founder can do with AI vs. what must be paid, and actionable 8-week MVP build timeline

---

## 1. EXECUTIVE SUMMARY

JETTO can be built in **three budget tiers**. The ideal path is **Tier B (Real MVP)**, which balances speed and capital efficiency. Founder should handle strategy, sales, and pilot management; contract dev builds the product with AI assistance.

**Budget Tiers:**
- **Tier A (Prototype-Only):** €0–500 | Figma + landing page | No working product
- **Tier B (Real MVP):** €6k–12k upfront + €30–70/mo infra | Fundable, pilot-ready
- **Tier C (Overbuild):** €20k+ | Unnecessary for MVP; avoid

**Recommended Path:** Raise €80k seed → allocate €35k to product (Tier B build + 6 months iteration) → €45k to ops, GTM, founder runway.

---

## 2. TIER A: PROTOTYPE-ONLY (€0–500)

### What You Get
- Figma clickable prototype (6-step loop: upload → comment → revise → ready → execute → archive)
- Landing page (Next.js + Vercel free tier) with waitlist signup
- Pitch deck (slides + one-pager)
- No backend, no real drawing upload, no live pilot capability

### Costs Breakdown
| **Item** | **Cost** | **What Founder Does** | **What Must Be Paid** |
|---|---|---|---|
| **Figma Design** | €0 | Founder uses Figma (free tier) + AI prompts (Claude/ChatGPT) to generate screens | None |
| **Landing Page** | €0–100 | Founder builds Next.js page with AI; Vercel free tier | €0 (free) or €50–100 (domain + premium template) |
| **Pitch Deck** | €0 | Founder writes (using AI for structure/copy) | None |
| **Total** | **€0–500** | — | — |

### What Founder Can Do With AI
- **Figma prototype:** Use ChatGPT/Claude to generate user flow descriptions → manually build in Figma (free design tool, no code)
- **Landing page:** Use v0.dev, Claude Artifacts, or Cursor to generate Next.js code → deploy to Vercel free tier
- **Waitlist backend:** Airtable (free) or Google Sheets + Zapier (free tier)
- **Pitch deck:** AI (Claude/ChatGPT) writes slide copy; founder formats in Google Slides

### Limitations
- **No working product:** Cannot pilot with real architects
- **Not fundable:** Investors want to see live usage, not Figma mocks
- **Use case:** Pre-product validation only (test landing page interest)

**Verdict:** Insufficient for serious fundraising or pilots. Skip unless bootstrapping without any capital.

---

## 3. TIER B: REAL MVP (€6K–12K + €30–70/MO) [RECOMMENDED]

### What You Get
- **Fully functional MVP:** Upload, version control, comment, notify, role permissions
- **Mobile-optimized:** Works on iOS/Android (PWA or React Native)
- **Cloud backend:** Supabase or Firebase (auth, DB, storage, notifications)
- **Pilot-ready:** 6 architects can use with real projects by Month 6
- **Iteration budget:** 6 months post-launch support (bug fixes, UX tweaks)

### Costs Breakdown

| **Item** | **Cost** | **What Founder Does** | **What Must Be Paid** |
|---|---|---|---|
| **Contract Developer** | €5,000–10,000 | Founder writes MVP spec (using AI), manages weekly sprints, tests builds | **Contract dev (8 weeks build + 12 weeks iteration)** |
| **Supabase/Firebase** | €0–50/mo (pilot), €50–100/mo (launch) | Founder sets up account, defines schema with AI help | **Cloud hosting subscription** |
| **PDF Rendering Library** | €0 (PDF.js open-source) or €200–500 (PSPDFKit license if needed) | Founder evaluates options with dev | **PSPDFKit license (if required)** |
| **Email Notifications** | €0–20/mo (SendGrid free tier = 100 emails/day; Resend = €0–20/mo) | Founder writes email templates with AI | **SendGrid/Resend subscription (if >100 emails/day)** |
| **Domain + SSL** | €15/year | Founder buys domain | **Domain registration** |
| **Figma/Design Tools** | €0 (Figma free tier sufficient for MVP) | Founder or dev designs screens | None |
| **WhatsApp Business API** | €0 (free tier: 1,000 conversations/mo) or €50–100/mo (if scaling) | Founder integrates with dev | **Meta Business API (post-pilot)** |
| **Total (One-Time)** | **€5,000–11,000** | — | — |
| **Total (Monthly, Pilot Phase)** | **€30–70/mo** | — | — |
| **Total (Monthly, Post-Launch)** | **€100–200/mo** | — | — |

### Developer Options

**Option 1: Contract Developer (€5k–10k for 8-week build + 12-week iteration)**
- **Scope:** Build MVP per spec; 20–30 hours/week for 8 weeks (build) + 10 hours/week for 12 weeks (iteration/bug fixes)
- **Profile:** Mid-level full-stack dev (React/React Native + Supabase/Firebase)
- **Cost:** €25–40/hour × 160–240 hours build + 120 hours iteration = **€7k–14k total**
- **Where to find:** Upwork (Italy-based devs), Toptal (higher quality, €40–60/hour), local Palermo dev community

**Option 2: Technical Co-Founder (Equity-Only)**
- **Cost:** €0 upfront; 20–30% equity
- **Pros:** Long-term commitment, shared ownership
- **Cons:** Slower fundraising (investors prefer single founder or clear CEO/CTO split); risk of co-founder breakup

**Recommended:** Contract dev for MVP, hire full-time CTO post-traction if needed.

### What Founder Can Do With AI (Save €2k–5k)

| **Task** | **Without AI** | **With AI (Founder + Claude/Cursor)** | **Savings** |
|---|---|---|---|
| **MVP Spec** | Hire product manager (€2k–5k) | Founder writes spec using AI (this document is example) | **€2k–5k** |
| **Database Schema** | Dev designs from scratch (8–10 hours) | Founder + AI draft schema; dev reviews/refines (2 hours) | **€200–400** |
| **Landing Page** | Dev builds (10 hours) | Founder builds with v0.dev/Cursor (2 hours founder time) | **€300–500** |
| **Email Templates** | Dev writes (4 hours) | Founder writes with ChatGPT (30 min) | **€100–150** |
| **User Stories** | Product manager writes (€1k–2k) | Founder + AI generate (this doc = AI-assisted) | **€1k–2k** |
| **Pitch Deck** | Hire consultant (€1k–3k) | Founder + AI write (this deck = AI-generated) | **€1k–3k** |
| **Total Savings** | — | — | **€4.6k–11k** |

**Key Insight:** Founder with AI can replace €5k–10k of product/marketing work, leaving dev to focus purely on code.

### Infrastructure Costs (Monthly)

| **Service** | **Pilot (18 projects)** | **Launch (120 projects)** | **Year 2 (300 projects)** |
|---|---|---|---|
| **Supabase** | €0 (free tier: 500MB DB, 1GB storage) | €25/mo (Pro: 8GB DB, 100GB storage) | €100/mo (Team: unlimited) |
| **Email (SendGrid/Resend)** | €0 (free: 100 emails/day) | €15–20/mo (40k emails/mo) | €50–80/mo (100k emails/mo) |
| **WhatsApp Business API** | €0 (free: 1k conversations/mo) | €50–100/mo (10k conversations) | €200–300/mo (50k conversations) |
| **Domain + SSL** | €1.25/mo (€15/year) | €1.25/mo | €1.25/mo |
| **CDN (CloudFlare)** | €0 (free tier) | €0 (free tier sufficient) | €20/mo (Pro: faster Italy delivery) |
| **Total Monthly** | **€1–30/mo** | **€91–146/mo** | **€371–501/mo** |

**Key Insight:** Infra costs are **negligible** during pilot (€0–30/mo), allowing founder to test product-market fit before spending on scale.

---

## 4. TIER C: OVERBUILD (€20K+) [AVOID FOR MVP]

### What This Buys (Unnecessarily)
- Full-time senior developer (€4k–6k/month × 4 months = €16k–24k)
- Custom backend (Node.js/PostgreSQL instead of Supabase) = +2–3 weeks dev time
- Desktop-native app (Electron) = +4 weeks dev time
- Advanced analytics dashboard (charts, insights) = +2 weeks dev time
- Automated payment processing (Stripe integration) = +1 week dev time

### Why to Avoid
- **Feature bloat:** MVP should be minimal; pilots don't need analytics dashboards
- **Slower iteration:** Custom backend = more maintenance, fewer AI shortcuts
- **Capital inefficiency:** Spending €20k on MVP leaves less for GTM, pilots, founder runway
- **Premature optimization:** Desktop app, analytics, payments are post-traction features

**Verdict:** Save this budget for post-Series A scale. For MVP, Tier B is sufficient.

---

## 5. WHAT FOUNDER CAN DO WITH AI (Detailed)

### 5.1 Product Specification (This Document = AI-Assisted)
- **Input:** Founder describes problem, ICP, workflows in natural language
- **AI (Claude/ChatGPT):** Generates user stories, data model, API endpoints, edge cases
- **Output:** MVP spec ready to hand to developer (saves €2k–5k product manager cost)

### 5.2 Database Schema Design
- **Input:** Founder describes entities (projects, drawings, comments, users)
- **AI:** Generates PostgreSQL schema with foreign keys, constraints, indexes
- **Output:** Schema.sql file ready for Supabase (saves 6–10 hours dev time = €200–400)

### 5.3 Landing Page + Waitlist
- **Tools:** v0.dev (Vercel AI), Cursor, Claude Artifacts
- **Input:** "Build a landing page for JETTO, a construction drawing vault. Hero section, problem, solution, waitlist form. Italian + English."
- **Output:** Next.js page deployed to Vercel in <2 hours (saves €300–500 dev cost)

### 5.4 Email Templates
- **Input:** "Write notification email for 'New drawing uploaded': Subject, body, call-to-action. Tone: professional, concise. Italian + English."
- **Output:** HTML email templates (saves €100–200 dev time)

### 5.5 Pitch Deck & One-Pager
- **Input:** Founder provides problem, solution, ICP, business model
- **AI:** Generates slide copy, speaker notes, competitive positioning (this deck = AI-generated)
- **Output:** Investor-ready deck in Markdown → convert to Google Slides/PowerPoint (saves €1k–3k consultant)

### 5.6 User Testing Scripts
- **Input:** "Generate 5 user testing scenarios for JETTO pilot architects. Focus on upload, comment, revision flows."
- **Output:** Testing scripts for pilot feedback sessions (saves €500–1k UX research cost)

### What AI CANNOT Do (Must Pay Developer)
- **Write production code** (AI generates boilerplate; dev refines, debugs, optimizes)
- **Mobile performance optimization** (PDF rendering, touch responsiveness)
- **Real-time notifications** (Supabase Realtime integration requires expertise)
- **GDPR compliance review** (AI can suggest, but legal/dev validation needed)
- **Security hardening** (authentication, role-based access control logic)

**Key Principle:** AI is a **force multiplier** for founder's time, not a developer replacement. Use AI to **draft, template, prototype**; pay dev to **build, optimize, secure**.

---

## 6. 8-WEEK MVP BUILD PLAN (TIER B)

### Pre-Dev (Week 0): Founder Preparation
- [ ] Write MVP spec using AI (this document)
- [ ] Create Figma wireframes (low-fidelity, AI-assisted)
- [ ] Set up Supabase project (free tier)
- [ ] Draft database schema with AI; review with dev candidate
- [ ] Hire contract developer (Upwork/Toptal)

### Week 1–2: Foundation
**Developer Tasks:**
- [ ] Supabase setup: PostgreSQL schema deployed, auth configured
- [ ] Basic Next.js/React Native app scaffold
- [ ] User signup/login flow (email/password)
- [ ] Create project flow (form → DB insert)

**Founder Tasks:**
- [ ] Test auth flow on mobile + desktop
- [ ] Write onboarding copy (AI-assisted)
- [ ] Draft email notification templates

**Milestone:** Authentication + project creation working

---

### Week 3–4: Drawing Upload + Version Control
**Developer Tasks:**
- [ ] File upload to Supabase Storage (PDF, DWG, JPG, PNG)
- [ ] Drawing entity creation (metadata + storage URL)
- [ ] Version logic: new upload marks old `is_latest = false`, increments version number
- [ ] List drawings (project detail page)

**Founder Tasks:**
- [ ] Upload test drawings (real construction PDFs from pilot architects)
- [ ] Verify version archiving works (old versions hidden, accessible via "See older versions")

**Milestone:** Upload + versioning functional

---

### Week 5–6: Comments + Notifications
**Developer Tasks:**
- [ ] PDF viewer integration (PDF.js or PSPDFKit)
- [ ] Comment anchoring: tap PDF → save (x, y, page, version_id)
- [ ] Comment thread UI (show/hide, resolve)
- [ ] Email notifications: SendGrid/Resend integration (new upload, comment, status change)

**Founder Tasks:**
- [ ] Test PDF rendering on iOS Safari, Android Chrome, desktop
- [ ] Verify email notifications received (check spam folder)
- [ ] Write Italian + English notification copy

**Milestone:** PDF commenting + email notifications live

---

### Week 7: Role Permissions + Drawing Status
**Developer Tasks:**
- [ ] Role-based access control (architect, PM, builder, owner permissions matrix)
- [ ] Drawing status transitions (draft → ready → executed)
- [ ] "Assign drawing" flow (PM/architect assigns to builder)
- [ ] Dashboard filters: "Assigned to Me", "Ready for Execution", "Executed"

**Founder Tasks:**
- [ ] Test permissions (create 4 test accounts: architect, PM, builder, owner)
- [ ] Verify builder cannot upload, owner cannot change status

**Milestone:** Permissions + status lifecycle complete

---

### Week 8: Mobile UI Polish + Pilot Prep
**Developer Tasks:**
- [ ] Mobile UX optimization (touch targets, responsive layout)
- [ ] Search functionality (by title, status, area)
- [ ] Audit trail page (view-only)
- [ ] Bug fixes from founder testing

**Founder Tasks:**
- [ ] User acceptance testing (UAT) with 2 pilot architects
- [ ] Collect feedback; prioritize Week 9+ fixes
- [ ] Prepare pilot onboarding materials (tutorial video, FAQs)

**Milestone:** MVP pilot-ready; deploy to production

---

### Week 9–20: Pilot Iteration (12 Weeks)
**Developer Tasks (10 hours/week):**
- [ ] Bug fixes from pilot feedback
- [ ] Minor UX improvements (e.g., faster PDF load, clearer notification copy)
- [ ] WhatsApp invite link generation (deep link)
- [ ] Export drawing + comments as PDF (Premium feature prototype)

**Founder Tasks:**
- [ ] Weekly check-ins with 6 pilot architects (3 projects each = 18 live projects)
- [ ] Iterate onboarding based on real usage
- [ ] Document case studies ("Before JETTO / After JETTO")

**Milestone (Week 20 / Month 5):** 18 active pilot projects; <5 critical bugs; pilot architects willing to recommend JETTO

---

## 7. FUNDABLE DEMO DEFINITION

A "fundable demo" is **NOT** a Figma prototype. Investors (especially pre-seed/seed) want to see **real usage**.

### Minimum Criteria for Fundable Demo
- [ ] **Live product:** Real drawing upload, comment, version control working
- [ ] **Real users:** ≥3 pilot architects using JETTO for ≥1 real project each (not test/dummy projects)
- [ ] **Weekly usage:** Pilot users logging in ≥2×/week; uploading drawings, commenting, marking status
- [ ] **Mobile-optimized:** Works on iOS/Android without critical bugs
- [ ] **Metrics to show investors:**
  - X active projects (target: 10–18)
  - Y total drawings uploaded (target: 50–100)
  - Z comments made (target: 100–200)
  - Retention: % of invited users who return weekly (target: >60%)

### What Investors Want to Hear
1. **"We have 6 architects piloting across 18 projects."** (Proof of distribution: architect-led growth)
2. **"720 drawings uploaded in 3 months; 340 comments exchanged."** (Proof of engagement)
3. **"Builders return 3×/week on average."** (Proof of habit formation)
4. **"2 architects hit the 2-project limit and asked to upgrade."** (Proof of monetization trigger)

**Bottom Line:** Fundable demo = 3+ months of real pilot usage with quantifiable engagement metrics. Figma mocks don't count.

---

## 8. TIMELINE SUMMARY (REALISTIC)

| **Month** | **Activity** | **Budget Spent (Cumulative)** | **Output** |
|---|---|---|---|
| **0** | Founder prep: spec, schema, hire dev | €0–500 (Figma, domain) | Contract dev hired |
| **1–2** | Dev: auth, project creation, upload | €1,500–2,500 | Basic app working |
| **3** | Dev: version control, notifications | €3,000–5,000 | Upload + notify working |
| **4** | Dev: comments, PDF viewer | €4,500–7,500 | PDF commenting live |
| **5** | Dev: permissions, status, polish | €6,000–10,000 | MVP pilot-ready |
| **6–8** | Pilot: 6 architects, 18 projects | €6,000–10,000 (+€30–70/mo infra) | Real usage begins |
| **9–12** | Iteration: bugs, UX, case studies | €7,000–12,000 (+€100–200/mo infra) | Fundable demo ready |

**Total (Year 1):** €7k–12k one-time + €500–1,200 infra = **€7.5k–13.2k total**

**With €80k Seed Round:**
- Allocate €35k to product (€12k MVP + €15k for 6 months post-launch dev + €8k buffer)
- Allocate €45k to ops, GTM, founder runway

---

## 9. RECOMMENDED ALLOCATION (€80K SEED)

| **Category** | **Amount** | **What It Buys** |
|---|---|---|
| **Product Development** | €35,000 | Contract dev (8 weeks MVP + 24 weeks iteration), Supabase Pro (12 months), PSPDFKit license, WhatsApp API, email service |
| **Pilot Operations** | €15,000 | 6 architect pilot incentives (€1k each = €6k), travel to pilot sites (€3k), localization/translation (€2k), user testing (€2k), design polish (€2k) |
| **Go-to-Market** | €20,000 | Landing page + SEO (€2k), case study production (€3k), local PR/media (€4k), architect outreach/events (€5k), WhatsApp invite tooling (€2k), contingency (€4k) |
| **Founder Runway** | €10,000 | 6–8 months minimal living expenses (€1,200–1,600/mo) while founder does sales, support, strategy full-time |
| **Total** | **€80,000** | 12-month runway to traction (120 projects, €1.8k MRR) |

---

## 10. BOOTSTRAP SCENARIO (NO FUNDING)

If founder cannot raise €80k, here's the **default-alive path**:

### Month 0–3: Prototype + Validation (€500)
- Build Figma prototype + landing page (founder + AI)
- Collect 50–100 waitlist signups
- Validate with 3–5 architect interviews
- **Cost:** €500 (domain, hosting, tools)

### Month 4–9: Solo Build (€2k–3k)
- Founder learns to code (free: freeCodeCamp, YouTube, ChatGPT tutoring)
- Build MVP using Cursor + Supabase + AI assistance (slower, but €0 dev cost)
- Launch pilot with 2–3 friendly architects (free tier)
- **Cost:** €2k–3k (Supabase, email service, pilot travel)

### Month 10–15: Paid Pilot (€3k–5k)
- Hire part-time dev (€2k–3k) to fix critical bugs founder can't solve
- Charge pilot architects €5–10/month (prove people will pay)
- Expand to 10–15 projects
- **Cost:** €3k–5k (dev, infra scale-up)

### Month 16–18: Raise or Go Ramen-Profitable
- If metrics strong (50+ projects, €300+ MRR): raise €50k–100k seed
- If metrics weak: continue bootstrapping until ramen-profitable (€1.5k–2k MRR = founder survival)

**Total Bootstrap Cost (18 months):** €5.5k–8.5k  
**Founder Time:** Full-time (no salary) for 12–18 months

**Trade-Off:** Slower (18 months vs. 12 months funded), higher risk (founder burnout, competitive threats), but zero dilution.

---

## 11. KEY COST OPTIMIZATION STRATEGIES

1. **Use AI Everywhere Founder Touches:** Spec writing, schema design, email templates, pitch deck, landing page → save €5k–10k
2. **Supabase Free Tier for Pilots:** Supports 500MB DB, 1GB storage = enough for 10–20 projects → save €300/year
3. **Email Free Tier (SendGrid):** 100 emails/day = 3,000/month = enough for 30–50 active users → save €240/year
4. **No Desktop App:** PWA works on desktop browsers; skip Electron → save €3k–5k dev time
5. **Manual Payment (Pilot Phase):** Invoice architects manually via PayPal/bank transfer; skip Stripe integration → save 1 week dev = €1k–1.5k
6. **Local Pilot Only (No Ads):** Recruit 6 architects in-person (Palermo architecture community); skip Facebook/Google ads → save €2k–5k
7. **Founder Does QA:** Founder tests all features instead of hiring QA engineer → save €2k–4k

**Total Potential Savings:** €13.5k–30.5k (brings Tier B down from €12k to €6k–8k if executed perfectly)

---

## 12. FINAL RECOMMENDATION

**For Funded Path (Recommended):**
- Raise €80k seed
- Allocate €35k to product (Tier B build + iteration)
- Hire contract dev (€25–40/hour, 280–400 hours over 8 months)
- Founder focuses on pilots, sales, case studies (AI-assisted)
- Target: 120 projects, €1.8k MRR by Month 12 → raise Series A or go default-alive

**For Bootstrap Path:**
- Spend €6k–8k over 12 months
- Founder builds MVP with AI + Cursor (slower, but feasible)
- Hire part-time dev for critical bugs only
- Target: 30–50 projects, €500–800 MRR by Month 12 → continue bootstrapping or raise small angel round

**DO NOT:**
- Spend €20k+ on MVP (overkill; save for scale)
- Build desktop-native app (PWA is sufficient)
- Hire full-time dev pre-traction (contract dev is cheaper + flexible)
- Ignore AI tools (they can save €5k–10k in product/marketing work)

---

**END OF COSTS DOCUMENT**  
*Use this to plan budget, hire developers, and present realistic financials to investors.*
