# Mosaic AI System — Long-Term Architecture Roadmap

*Maintained by Reginald Pemberton IV. Last updated: May 2026.*

---

## Agent Architecture

```
Ben Tinklenberg
      │
      ├── Reginald Pemberton IV  [LIVE]
      │     Personal life · household · accountability
      │     Morning ritual · goal tracking (Ben + Emily)
      │     Interface: Telegram (Ben + Emily)
      │     Home: /root/.hermes · hermes-gateway.service
      │
      ├── Clara Voss  [LIVE]
      │     Email triage · permit tracker · consultant manager
      │     Reads ben@mosaicdev.co · sends with approval only
      │     Interface: Telegram (Ben only)
      │     Home: /root/.hermes-clara · hermes-clara.service
      │
      ├── Vera Chen  [PLANNED]
      │     Zoning research · due diligence automation
      │     Monitors new active deals · researches jurisdiction zoning
      │     Writes findings back to screener DB
      │     No Telegram (background agent)
      │     Home: /root/.hermes-vera · hermes-vera.service
      │
      ├── Margot  [PLANNED]
      │     Social media strategist · content production
      │     Drafts LinkedIn posts and X tweets for Ben's voice
      │     Monitors completed deals and project milestones
      │     Interface: Telegram (Ben only)
      │     Home: /root/.hermes-margot · hermes-margot.service
      │
      ├── Jack (Contractor Scout)  [PLANNED]
      │     Local contractor outreach · lead generation
      │     Monitors Silicon Valley Facebook groups, community pages
      │     Drafts RFPs and initial outreach messages
      │     Interface: Telegram (Ben only)
      │     Home: /root/.hermes-jack · hermes-jack.service
      │
      └── Alfred Quartersworth  [PLANNED]
            Business CoS · orchestrates Mosaic automation
            Interface: Telegram (Ben + Donovan)
            Home: /root/.hermes-alfred · hermes-alfred.service
                  │
                  └── Mosaic department agents (Phases 1–5)
                        Deal Scout · Underwriter · Permit Tracker
                        DD Agent · Project Ops
```

---

## Repository Structure

| Repo | Purpose | Primary writer |
|------|---------|----------------|
| `bentink/mosaic-company-os` | Company OS — strategy, systems, decisions, agent architecture | Alfred |
| `bentink/mosaic-project-memory` | Live project memory — status, permits, contacts, action items, logs | Clara |
| `bentink/mosaic-deal-screener` | Deal screening and underwriting engine | Claude Code |
| `bentink/cyborg-mode-cos` | Reginald backend API (tasks, goals, dashboard) | Claude Code |

---

## File Structure — mosaic-company-os

```
mosaic-company-os/
├── COMPANY.md          # Source of truth — identity, team, strategy, systems
├── ROADMAP.md          # This file — agent architecture and long-term plan
├── DECISIONS.md        # Decision log — significant calls with rationale
└── SYSTEMS.md          # System specs (when COMPANY.md gets too long)
```

---

## File Structure — mosaic-project-memory

```
mosaic-project-memory/
├── README.md                          # Active project index
├── CLARA.md                           # Clara's operating instructions
├── templates/
│   └── project/                       # Copy this to scaffold a new project
│       ├── README.md
│       ├── status.md
│       ├── action-items.md
│       ├── contacts.md
│       ├── permits.md
│       └── logs/
└── projects/
    └── <slug>/                        # One directory per active project
        ├── README.md                  # Address, jurisdiction, APN, overview
        ├── status.md                  # Current phase, next actions
        ├── action-items.md            # Open items with owner and due date
        ├── contacts.md                # City planners, consultants, utilities
        ├── permits.md                 # Permit tracker — all applications
        └── logs/
            └── YYYY-MM-DD.md         # Clara's daily check-in log
```

---

## Google Drive Structure

Root folder (Clara-accessible): `1j9ghT_KNmAugmgi6keVfXRF6gtx99WMk`

```
Mosaic Project Documents/
└── <Project Name>/
    ├── Contracts/
    ├── City Correspondence/
    ├── Plans & Drawings/
    ├── Reports/
    │   ├── Geotechnical/
    │   ├── Environmental/
    │   └── Title/
    ├── Permits/
    │   ├── Applications/
    │   └── Approvals/
    ├── Financials/
    │   ├── Underwriting/
    │   └── Fee Receipts/
    └── Photos/
```

Clara reads and writes only within this folder. She does not touch the admin folder or any other Drive content.

---

## Infrastructure

| Service | Description | Location |
|---------|-------------|----------|
| `hermes-gateway.service` | Reginald — personal agent gateway | Hetzner 5.78.222.187 |
| `hermes-clara.service` | Clara — email triage agent gateway | Hetzner 5.78.222.187 |
| `hermes-alfred.service` | Alfred — business CoS gateway (planned) | Hetzner 5.78.222.187 |
| `reginald.service` | FastAPI data backend for Reginald dashboard | Hetzner 5.78.222.187 |
| Mosaic Deal Screener | Deal pipeline and underwriting | Railway |

**Doppler projects:**
- `reginald` — Reginald + backend secrets
- `clara` — Clara's secrets (isolated)
- `alfred` — Alfred's secrets (planned, isolated)

**Domain:** `reginald.mosaicdev.dev` → Hetzner

---

## Build Status

### Live
- Reginald — Telegram, memory, crons, Google Calendar (ben@mosaicdev.co)
- Clara — Telegram bot, Gmail read/send, Drive (scoped), project memory (GitHub)
- Mosaic Deal Screener — PropertyRadar webhooks, underwriting, pipeline dashboard

### In Progress
- Reginald dashboard UI (Ben building frontend, API endpoint needed)
- mosaic-project-memory scaffolding (406 7th Ave active)

### Planned — Near Term
- Clara email triage cron (fires when Ben messages Clara on Telegram)
- Clara portal status checking (San Mateo County Accela)
- Clara Drive document filing (save city correspondence to correct project folder)
- Vera Chen — zoning research agent (polls screener DB hourly, researches jurisdiction zoning)
- Jack (Contractor Scout) — monitors Facebook groups, drafts contractor outreach
- Alfred Quartersworth (business CoS — fork of Reginald stack)
- Mosaic deal screener as MCP tool callable by Alfred

### Planned — Medium Term
- Alfred: deal flow orchestration — off-market lead sourcing, broker outreach CRM
- Alfred: underwriting memo generator
- Alfred: LP update drafter
- Server hardening (SSH password auth off, fail2ban, UFW)
- HTTPS / SSL for reginald.mosaicdev.dev
- Narrow Clara's Drive OAuth scope to `drive.file` (currently full Drive)

### Planned — Long Term
- Company OS licensed to other firms — config-driven repeatable deployment
- Department agents as MCP servers: Deal Scout, Underwriter, DD, Permit, Project Ops
- iMessage integration (requires BlueBubbles on a Mac host)

---

## Infrastructure Principles

1. **Clean separation.** Reginald handles the person. Clara handles permits and email. Alfred handles the company. They do not share context promiscuously.
2. **Isolated secrets.** Each agent has its own Doppler project. No cross-contamination.
3. **Config-driven and repeatable.** Everything built for Mosaic must be packageable for another firm.
4. **No unnecessary dependencies.** Build or use local credentials unless a dependency provides clear, sustained value.
5. **Everything surfaces through Telegram.** No new apps, no new habits required.
6. **Approval before action.** Clara never sends email autonomously. Alfred never posts or transacts without approval.

---

*Reginald Pemberton IV, CoS — Tinklenberg household*

