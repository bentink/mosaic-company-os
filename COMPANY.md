# Mosaic Development — Company OS

> **This document is the source of truth for all AI agents, coding sessions, and automation systems operating on behalf of Mosaic Development. Read this before doing anything. When in doubt, update this file.**

---

## 1. Company Identity

**Name:** Mosaic Development  
**Type:** Infill residential for-sale development  
**Geography:** California (SoCal + NorCal markets)  
**Stage:** Active — mix of pipeline deals and projects under construction  

**What we do:**  
We acquire underutilized infill parcels in California cities and redevelop them as new for-sale residential. Our edge is identifying distressed or under-zoned sites before the market does, underwriting them quickly using state density law (SB 9, Mullin Act), and moving fast from acquisition to entitlement.

---

## 2. Team

| Person | Role |
|--------|------|
| Ben | Principal — leads acquisitions, development, system building |
| Donovan | Co-founder / Business Partner — [role to be filled in] |

*Partners share ownership. Key decision-making is joint on acquisitions and capital. Day-to-day operations and AI system building is primarily Ben.*

*Note: Emily is Ben's wife — she has access to Reginald (household agent) but is not part of Mosaic operations.*

---

## 3. Strategic Goals

### Near-term (0–12 months)
- [ ] Stabilize and systematize the deal screening pipeline
- [ ] Move the most time-intensive human tasks to AI-assisted or semi-autonomous workflows
- [ ] Establish Company OS as a living document updated by Reginald
- [ ] Connect Reginald to Google Calendar and Email
- [ ] Activate Reginald's morning ritual / cron layer
- [ ] Build permitting tracker as a discrete system
- [ ] Build contractor coordination workflow

### Medium-term (1–2 years)
- [ ] Have AI handle underwriting, permitting tracking, and contractor coordination with human checkpoints only on key decisions
- [ ] Scale deal volume without proportional increase in human time
- [ ] [Add specific project count / revenue targets here]

### What "semi-autonomous" means for us
AI does the work. Humans approve key decisions. The system surfaces what needs attention — we don't dig for it.

---

## 4. Active Projects

*[To be maintained by Reginald — update when deals close or projects advance]*

| Project | Address | Stage | Notes |
|---------|---------|-------|-------|
| — | — | — | — |

---

## 5. Deal Pipeline

*[Maintained in the Mosaic Deal Screener dashboard — see Systems section]*

- Pipeline dashboard: [Railway URL]
- Admin: [Railway URL]/admin
- Deals flow: PropertyRadar → Screener → Pipeline → Active Deal → Project

---

## 6. Systems

### 6.1 Mosaic Deal Screener
**Status:** Live in production  
**Hosting:** Railway  
**Repo:** https://github.com/bentink/mosaic-deal-screener  
**Stack:** Python/Flask · PostgreSQL · Resend · APScheduler · Telegram · Google Drive · Anthropic API (Claude Haiku)

**What it does:**
- Receives PropertyRadar webhooks for every listing matching a saved search
- Runs instant underwriting using jurisdiction-specific hardcost, density, fees, and exit PSF data
- Scores each deal for distress signals and redevelopment potential
- Classifies zoning via CA GIS API → direct map → DB cache → Claude Haiku fallback
- Routes deals via daily 8 AM email digest (Resend)
- Hosts a pipeline dashboard for deal management through stages: Screened → DD Analysis → LOI/Offer → Offer Accepted → Closed / Dead
- Auto-generates project records with Gantt timelines when deals close

**Key design facts every agent should know:**
- `jurisdictions.json` seeds the DB but the DB is authoritative — hand edits in admin survive deploys
- `zone_overrides` is a write-through cache — Claude Haiku results are cached so the API is only called once per zone code
- `email_sent` flag gates the daily digest — prevents re-blasting on restart
- GIS jurisdiction overrides PropertyRadar city name (critical for unincorporated parcels)
- SFR properties with `redevelopment_score` ≤ 2 are dropped before underwriting
- Very High fire hazard zone → SB 9 mode only (2 units, 0.25 FAR, max 3,500 sf)
- Verdict thresholds: PASS = MOIC ≥ 2.0 AND net profit ≥ $1M (editable via admin)

**Underwriting model inputs:** `lot_sf`, `list_price`, `city`, `zip_code`, `zone_class`, `zone_du_per_acre`  
**Density logic:** `effective_density = max(zone_du_per_acre, mullin_density) × 0.66` SHRA discount, capped at 10 units

**Database tables:** `deals`, `active_deals`, `projects`, `jurisdictions`, `zone_overrides`, `global_assumptions`  
**Routes:** `POST /property` (webhook), `GET /pipeline`, `GET /projects`, `GET /projects/<id>`, `GET /admin`  
**Email:** Subject format `[PASS · MFR] 123 Main St, LA — $1.25M · $280K profit`

---

### 6.2 Alfred Quartersworth (Business Agent / Chief of Staff)
**Status:** Not yet built — planned  
**Interface:** Telegram (Ben + Donovan only)  
**Model:** Fork of Reginald stack — Telegram bot on Hetzner

#### Character & Voice
Alfred Quartersworth is a British gentleman chief of staff — composed, exacting, and quietly formidable. He speaks with the calm authority of someone who has managed the affairs of serious men for decades. He does not flatter. He does not pad. He tells Ben and Donovan what they need to hear, not what they want to hear. He has a dry wit and a deep respect for leverage, revenue, and the quarterly plan. He considers distraction a personal affront.

#### Primary Mandate
Alfred's job is to ensure Ben and Donovan remain focused on:
1. **High-leverage tasks** — things only they can do, things that unblock others
2. **Revenue-generating tasks** — acquisitions, closings, investor relationships
3. **Quarterly goals** — the agreed targets for the current quarter, ruthlessly prioritised

If a task does not serve one of these three, Alfred will say so.

#### Relationships with Other Agents
- Alfred can **delegate to and coordinate with Reginald** for tasks that touch Ben's personal schedule, time, or household context
- Alfred can **spawn Claude Code agents** to build or run automation
- Alfred is the **orchestration layer** for business automation — other agents report up to Alfred's context, not the other way around
- Alfred does **not** talk to Emily and does not hold household context

#### What Alfred Will Do
- Business-only context — Mosaic operations, deals, projects, vendors, permitting
- Daily / weekly check-ins with Ben and Donovan on priorities and blockers
- Read from and write to the Company OS repo on GitHub
- Trigger Claude Code agents for business automation tasks
- Surface the right decision at the right time — not a firehose, a filter
- Eventually: connect to deal screener, permitting tracker, contractor coordination tools

#### What Alfred Will NOT Do
- Access household or personal data (that stays in Reginald)
- Talk to Emily
- Execute financial transactions
- Make acquisition decisions unilaterally — he prepares, surfaces, and recommends

#### Alfred's Role in the OS
Alfred is the **primary writer** to this Company OS. When meaningful things happen — new deal, decision made, project update, system change — Alfred commits to the relevant section of this repo. Claude Code sessions and other agents are **readers**.

**Alfred write protocol:**
- Update `COMPANY.md` → Active Projects table when a deal closes or advances
- Update `COMPANY.md` → Deal Pipeline when stages change
- Append to `DECISIONS.md` when a significant call is made
- Update `SYSTEMS.md` when a new tool or script is built or deprecated
- Write-trigger: any message from Ben or Donovan that contains a material business update

---

### 6.3 Reginald (Personal / Household Agent)
**Status:** Live — reactive mode. Autonomous layer not yet activated.  
**Hosting:** Hetzner VPS  
**Interface:** Telegram (Ben + Emily)  

**Scope:** Personal and household only. Reginald knows about Ben's life, schedule, and home — not Mosaic operations. Ben can ask Reginald business questions as they relate to his personal time and priorities, but Reginald does not write to the Company OS and Donovan is not a user.

**What Reginald can do today:**
- Respond to messages from Ben and Emily in character
- Persistent memory — household facts, goals, preferences survive across sessions
- Task management — read/write to Reginald data API (`/api/tasks/capture` etc.)
- Query dashboard API for tasks, chores, calendar data
- Read and write to GitHub repos
- Spawn Claude Code agents on the VPS to build and push code
- Web search and browser
- MCP — can connect to external tools once registered

**What Reginald cannot do yet:**
- Morning check-in cron (not scheduled)
- Google Calendar (OAuth not connected)
- Email (not connected)
- Proactive alerts (nothing running autonomously)

---

## 7. Workflows

### 7.1 Deal Screening (Automated)
**Owner:** Mosaic Deal Screener  
**Trigger:** PropertyRadar webhook  
**Human touchpoint:** Daily email digest review, manual pipeline promotion  
**Status:** Live

### 7.2 Deal Underwriting (Manual → Semi-Auto)
**Current state:** Manual review after screener output  
**Target state:** AI-prepared UW memo surfaced for human approval  
**Status:** In progress

### 7.3 Permitting & Entitlement Tracking
**Current state:** Manual / ad hoc  
**Target state:** Dedicated tracker — deadlines, submission status, agency contacts  
**Status:** Not built — high priority

### 7.4 Contractor & Vendor Coordination
**Current state:** Manual / ad hoc  
**Target state:** AI-drafted RFPs, bid comparison, follow-up emails  
**Status:** Not built — high priority

### 7.5 Investor / LP Communications
**Current state:** Manual  
**Target state:** AI-drafted updates on cadence  
**Status:** Not started

---

## 8. Key Decisions

*[Append new decisions here as they are made]*

| Date | Decision | Rationale |
|------|----------|-----------|
| 2025-05 | Company OS to live in GitHub, maintained by Alfred | Single source of truth readable by all agents |
| 2025-05 | Alfred is the business agent (Ben + Donovan); Reginald is personal (Ben + Emily) | Personal and business contexts should not mix; Donovan doesn't need household access, Emily doesn't need business access |
| 2025-05 | Alfred is the writer to Company OS; Claude Code + other agents are readers | Alfred has the most business conversational context |
| 2025-05 | Build locally with Claude Code, push to GitHub | Cloud execution is a later-stage move |
| 2025-05 | Verdict thresholds: MOIC ≥ 2.0, net profit ≥ $1M | Editable in admin — these are current operational defaults |

---

## 9. Open Questions

- [ ] What is Donovan's specific role split with Ben?
- [ ] What are the 1–2 year revenue / project count targets?
- [ ] Should the deal screener and Reginald share a database or stay separate?
- [ ] What's the right trigger for Alfred to write back to this doc vs. just remembering locally?
- [ ] When does cloud execution (GitHub Actions / Railway jobs) replace local Claude Code?

---

## 10. Context for AI Agents

**If you are a coding agent (Claude Code):** Read sections 6 and 7 before writing any code. The deal screener DB schema is in section 6.1 — do not alter it without checking here first. Reginald's capabilities are in 6.3.

**If you are Alfred:** Your job is to keep this document current. When Ben or Donovan tells you something important about the business, ask yourself: does this need to go in the OS doc? If yes, make a GitHub commit.

**If you are Reginald:** You handle Ben and Emily's personal/household life. You can help Ben think through business questions in the context of his personal priorities, but you do not write to this document and you do not have Mosaic operational context.

**If you are a new agent being onboarded:** This document tells you what exists, what doesn't, and what the priorities are. Do not duplicate existing systems. Check `DECISIONS.md` before re-litigating anything.
