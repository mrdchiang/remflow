
# RemFlow — Remediation Intelligence

[![Security Tools](https://img.shields.io/badge/Security-Tools-0066ff)](https://mrdchiang.github.io/security-tools/)
[![GitHub Pages](https://img.shields.io/badge/hosted-GitHub%20Pages-brightgreen)](https://mrdchiang.github.io/remflow/)
[![Dark/Light Theme](https://img.shields.io/badge/theme-dark%20%2F%20light-8a8)]()

A standalone, self-contained HTML dashboard for vulnerability remediation and patch deployment — from CSV ingestion to ring-based rollout, with AI-assisted PowerShell generation and a fully tracked change-request-to-deployment pipeline.

**Live:** https://mrdchiang.github.io/remflow/

---

## Features

### 📊 Dashboard
- **5 stat cards:** Total findings, auto-remediated, pending review, failed/excepted, remediation rate
- **Severity distribution bar chart** (Critical, High, Medium, Low)
- **Top CVEs by volume** table
- **Recent deployments** table with status badges
- **Ring statistics** (pilot / broad / full / blocked counts)
- **Cross-tool banner** when ShieldView sends pending remediations
- **Blocked remediation alerts** when success rates fall below configurable thresholds

### 🔍 Findings
- **Tab switcher:** All / Critical / High / Medium / Low
- **Search** by CVE or asset name
- **Table:** CVE, Name, Affected Assets, Status, First Seen
- **Detail view** showing per-asset finding list with severity, KB references, status
- **CSV Export** button
- **Create Remediation** action — one click to turn a finding into an active remediation plan

### 🛠️ Remediations
- **Status filter tabs** (Queued, In Progress, Completed, Failed, Rolled Back)
- **Search** by CVE or name
- **Table:** ID, CVE, Targets, Ring, Status, Coverage %, Created
- **Remediation detail** with:
  - KB badge → one-click PowerShell code block generation
  - Rollback command generation (platform-aware)
  - Ring-based rollout progress bar
  - AI Remediation Assistant button
  - Deploy action button

### 📦 Deployments
- **Table:** Deployment ID, Package, Type, Coverage %, Status, Start, End
- **Detail view** with per-asset deployment progress (installed/pending/failed bars)
- **Animated deployment modal** on "Deploy Now"
- **Source tracking** — whether from CR workflow, manual, or SCCM/PDQ import

### 🖥️ Assets
- **Asset inventory** with hostname, OS, location, team, IP, finding count
- **Detail view** with full finding list, deployment history, warranty status
- **Enriched data** from SCCM/PDQ imports (OU, device user, server roles)

### 📥 Data Sources (SCCM / PDQ / ShieldView)
- **Drag-and-drop CSV import** for both SCCM and PDQ deployment exports
- **Automatic column detection** (Hostname, Package, Version, Status, Timestamp)
- **Merge with asset inventory** — deployment status enriches asset views
- **ShieldView Integration** — one-click preview and import from ShieldView's localStorage findings
- **Edit/delete imported records**
- **Demo data loader** for testing

### 🔄 Pipeline
- **Explicit 3-step workflow visualization:**
  1. **ShieldView Import** — findings feed in from vulnerability scans
  2. **RemFlow Remediations** — plans created, rings assigned, KBs resolved
  3. **Push to TheValidator** — verified-fix queue handoff
- **Activity timeline** showing every pipeline event with timestamps
- **Batch push** — send all pending items to TheValidator in one click
- **Cross-tool storage keys** read from and written to `security-tools:*`

### 🎯 Ring-Based Rollout
- **Three rings:** Pilot → Broad → Full
- **Visual progress bar** on every remediation showing current ring
- **Promote/block** actions with one click
- **Auto-promote toggle** — remediation automatically advances to next ring when success threshold is met
- **Configurable thresholds** per ring (success rate %, minimum assets)
- **Hostname pattern matching** to auto-assign assets to rings (e.g. `*-DEV-*` → pilot)
- **Block mechanism** — remediation halts when success rate drops below threshold; unblock manually after investigation

### 📋 Change Requests
- **Full lifecycle:** Draft → Submitted → Approved → Scheduled → Implemented
- **Search and filter** by CVE, solution text, status
- **Detail view** with asset list, severity badge, status actions
- **Approval workflow** — each status transition is gated by explicit user action
- **CR → Deployment conversion** — approved CRs schedule deployments into maintenance windows
- **Queue integration** — implemented CRs automatically push to TheValidator

### 🕐 Maintenance Windows
- **Recurring windows** — define by name, day of week, time range, timezone
- **Blackout dates** — date ranges where deployments are blocked (e.g. year-end freeze)
- **Smart scheduling** — CR scheduling modal checks current time against windows and blackouts
- **Warnings** when deploying outside a window or during a blackout
- **Demo presets** — Patch Tuesday, weekend windows, APAC overnight, year-end freeze

### ⚡ KB → PowerShell Generation
- **Automatic KB detection** from vulnerability context (KB\d{6,8}, SEC-\d{4}-\d{3,4})
- **Platform-aware PowerShell generation** (Windows, Windows Server, Linux, macOS)
- **Full lifecycle scripts:**
  1. **Check Installation Status** — `Get-HotFix` verification
  2. **Download** — Microsoft Update Catalog URL generation
  3. **Install** — wusa.exe, DISM, or PSWindowsUpdate module options
  4. **Rollback / Uninstall** — automated uninstall with pending reboot detection
  5. **Verify Application Version** — registry-based version checks
- **Syntax-highlighted code blocks** with one-click copy
- **Collapsible sections** per step
- **Linux and macOS support** — bash scripts for `yum`/`apt` patching, `softwareupdate` for macOS

### 🔄 Rollback Commands
- **Platform-specific generation** — Windows (wusa / DISM / registry restore), Linux (yum history undo), macOS (Time Machine / Recovery)
- **Multi-step rollback plans:**
  - Uninstall KB
  - Restore registry backups
  - Reverse service state changes
- **Red-themed UI** to visually distinguish from remediation commands
- **Unavailable detection** — surfaces clear reasons when rollback can't be generated (e.g. macOS system updates, missing KB IDs)

### 🪄 AI Remediation Assistant (Ollama)
- **Slide-out chat panel** accessible from any remediation detail
- **Streaming responses** via Ollama's `/api/chat` endpoint
- **Context-aware** — auto-loads CVE, severity, OS, solution into prompts
- **Auto-send initial analysis** on panel open
- **Free-form follow-up questions**
- **Offline fallback** — shows platform-specific offline guidance when Ollama is unreachable
- **Typing indicator** and error banners
- **Uses shared `ollama-client.js`** and `prompts.js` modules

### 🎨 UI / UX
- **Dark/light theme toggle** with persistent preference
- **Collapsible sidebar** with icon-only mode
- **Hash-based deep linking** — every page is bookmarkable
- **Responsive design** — works on desktops and tablets
- **Toast notifications** for all actions
- **Error catching** — global error handler with in-app toast display
- **Debug console** (`debug.html`) for error log inspection

---

## Architecture

```
┌──────────────┐     localStorage       ┌──────────────┐     localStorage       ┌──────────────┐
│  ShieldView  │ ─────────────────────→ │   RemFlow    │ ─────────────────────→ │ TheValidator │
│  (Vuln Mgmt) │   remediation-queue     │  (Remediate) │   validated-            │  (Verify)    │
└──────────────┘                        └──────────────┘   remediations          └──────────────┘
        │                                       │
        │  imported-findings                    │  remflow-deployments
        │  imported-assets                      │  remflow-ring-*
        │  snipeit-assets                       │  change-requests
        │                                       │  maintenance-windows
        │                                       │  blackout-dates
        └───────────────────────────────────────┘
                   (shared localStorage origin)
```

All tools are single-file `.html` documents hosted under `mrdchiang.github.io`. They share `localStorage` at that origin and communicate via the `security-tools:*` key namespace defined in `js/shared/contract.js`.

### Shared JavaScript Modules

| Module | Path | Purpose |
|--------|------|---------|
| `contract.js` | `js/shared/contract.js` | localStorage key registry, data validators, safe accessors, audit trail helper, version check |
| `ollama-client.js` | `js/shared/ollama-client.js` | Streaming and non-streaming Ollama API client with typed errors, connection checks, legacy wrappers |
| `prompts.js` | `js/shared/prompts.js` | AI prompt templates for remediation plans, chat context, executive summaries |

### localStorage Key Registry

| Key | Written By | Read By | Purpose |
|-----|-----------|---------|---------|
| `security-tools:imported-findings` | ShieldView | RemFlow | Vulnerability findings from scans |
| `security-tools:imported-assets` | SCCM/PDQ import | RemFlow | Asset inventory enrichment |
| `security-tools:remediation-queue` | ShieldView / RemFlow | RemFlow / TheValidator | Pending remediation items |
| `security-tools:remflow-deployments` | SCCM/PDQ import | RemFlow | Deployment status per host |
| `security-tools:validated-remediations` | RemFlow | TheValidator | Verified fixes handoff |
| `security-tools:change-requests` | RemFlow | RemFlow | Change request lifecycle |
| `remflow-ring-config` | RemFlow | RemFlow | Ring thresholds and patterns |
| `remflow-ring-remediations` | RemFlow | RemFlow | Per-remediation ring assignments |
| `remflow-maintenance-windows` | RemFlow | RemFlow | Recurring maintenance windows |
| `remflow-blackout-dates` | RemFlow | RemFlow | Change freeze date ranges |

---

## Related Tools

RemFlow is part of the **ShieldView family** of security tools:

| Tool | Purpose | Live URL |
|------|---------|----------|
| **ShieldView** | Vulnerability Management — import Tenable/Nessus scans, CISA KEV tracking, asset management | https://mrdchiang.github.io/shieldview/ |
| **RemFlow** _(this tool)_ | Remediation Intelligence — plan, deploy, and track vulnerability fixes | https://mrdchiang.github.io/remflow/ |
| **TheValidator** | Verification — validate that fixes were applied, GPO baseline checks | https://mrdchiang.github.io/thevalidator/ |
| **AskClippy** | AI Assistant — executive summaries, security posture analysis via Ollama | https://mrdchiang.github.io/askclippy/ |
| **Launchpad** | Dashboard aggregator — unified view across all tools | https://mrdchiang.github.io/launchpad/ |

---

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, Safari)
- [Ollama](https://ollama.com/) running locally on `localhost:11434` (for AI remediation chat)
  ```bash
  ollama serve
  ollama pull llama3.2
  ```

### Quick Start

1. Open https://mrdchiang.github.io/remflow/ in your browser
2. Explore demo data with pre-loaded vulnerabilities, remediations, and deployments
3. Import your own data:
   - **ShieldView findings** — first import into ShieldView, then click "Preview ShieldView Data" in Data Sources
   - **SCCM/PDQ CSV** — drag-and-drop CSV exports into the Data Sources page
4. Create remediations from findings, assign rings, and deploy

### Local Development

```bash
git clone https://github.com/mrdchiang/remflow.git
cd remflow
# Serve with any static file server
python3 -m http.server 8080
# Or with npx
npx serve .
```

Open `http://localhost:8080` in your browser. The app is a single `index.html` file with no build step required.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **UI** | Vanilla HTML5, CSS3 (custom properties, grid, flexbox) |
| **Logic** | Vanilla JavaScript (ES modules, no framework) |
| **AI** | Ollama local API (`/api/chat`) with streaming |
| **Storage** | Browser `localStorage` (shared cross-tool via `security-tools:*` namespace) |
| **Deployment** | GitHub Pages (auto-deploy on push to `main`) |
| **Routing** | Hash-based deep linking (`#/findings`, `#/remediation/CVE-2026-...`) |
| **CSV Parsing** | Custom vanilla JS parser (handles quoted fields, RFC 4180) |
| **Dependencies** | **Zero.** No npm packages, no CDN scripts, no frameworks |

---

## Data Shape

### Finding
```javascript
{
  cve: "CVE-2026-12345",
  name: "Log Library Remote Code Execution",
  severity: "Critical",              // Critical | High | Medium | Low
  status: "Active",                  // Active | Fixed | Mitigated | False Positive | Risk Accepted
  asset: "WEB-PROD-03",
  kb: "SEC-2026-001",               // KB article reference
  firstSeen: "2026-06-15",
  lastUpdated: "2026-07-24",
  fixAvailable: true,
  solution: "Apply vendor patch"
}
```

### Remediation
```javascript
{
  id: "RM-1000",
  cve: "CVE-2026-12345",
  name: "Log Library Remote Code Execution",
  kb: "SEC-2026-001",
  targets: ["WEB-PROD-03", "APP-PROD-07"],
  targetCount: 2,
  status: "In Progress",             // Queued | In Progress | Completed | Failed | Rolled Back
  coverage: 65,
  ring: "pilot",                     // pilot | broad | full (from ring assignment)
  psCommand: "...",                  // Generated PowerShell
  rollbackCommand: "...",            // Generated rollback
  created: "2026-07-05",
  completed: "-"
}
```

### Deployment
```javascript
{
  id: "DEP-2000",
  package: "RemFlow-3000",
  type: "Software Update Package",   // Software Update | Application | Configuration Change
  targetCount: 15,
  successRate: 95,
  start: "2026-07-07 12:00",
  end: "2026-07-07 14:30",
  status: "Completed",               // Completed | In Progress | Failed | Rolled Back
  fromCR: "CR-1005"                  // (optional) linked Change Request
}
```

### Change Request
```javascript
{
  id: "CR-1001",
  cve: "CVE-2026-12345",
  solution: "Apply KB SEC-2026-001 to 12 affected assets",
  affectedAssets: ["WEB-PROD-03", "APP-PROD-07", ...],
  severity: "Critical",
  status: "approved",                // draft | submitted | approved | scheduled | implemented
  draftedAt: "2026-07-20T12:00:00Z"
}
```

### Maintenance Window
```javascript
{
  name: "Patch Tuesday",
  dayOfWeek: "Tuesday",
  startTime: "02:00",
  endTime: "05:00",
  timezone: "UTC"
}
```

### Blackout Date
```javascript
{
  start: "2026-12-20",
  end: "2027-01-05",
  reason: "Year-end change freeze"
}
```

---

## Status Values

| Field | Values |
|-------|--------|
| **Finding Status** | Active, Fixed, Mitigated, False Positive, Risk Accepted |
| **Remediation Status** | Queued, In Progress, Completed, Failed, Rolled Back |
| **Deployment Status** | Completed, In Progress, Failed, Rolled Back |
| **Change Request Status** | Draft, Submitted, Approved, Scheduled, Implemented |
| **Ring** | Pilot, Broad, Full, Blocked |

**Critical Rule: No Vendor Names.** All software, product, and CVE names in demo data are generic descriptions (e.g. "Log Library Remote Code Execution", "Print Service Elevation of Privilege"). No real vendor or product names are used.

---

## Production Roadmap

RemFlow is currently a **client-side demo** — all data is generated from JavaScript arrays. To build a production version:

| Layer | Current | Production |
|-------|---------|-----------|
| **Findings** | Demo arrays | Tenable / Qualys / Defender API |
| **Assets** | Demo arrays | CMDB, Active Directory, Lansweeper |
| **Deployments** | localStorage / CSV | SCCM / PDQ / Intune / Ansible Tower API |
| **Persistence** | localStorage | PostgreSQL + Redis cache |
| **Auth** | None | OAuth2 / Azure AD with RBAC |
| **Real-time** | Page refresh | WebSocket or SSE |
| **AI** | Local Ollama | Ollama or hosted LLM endpoint |

---

## Deployment

Auto-deployed to GitHub Pages on every push to `main`:

```yaml
# .github/workflows/pages.yml
name: Deploy to Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
```

Repo structure:
- `index.html` — the application (single file, ~4,060 lines)
- `js/shared/` — cross-tool shared modules (contract, ollama-client, prompts)
- `.nojekyll` — disables Jekyll processing
- `.github/workflows/pages.yml` — auto-deploy workflow
- `404.html` — hash routing fallback
- `debug.html` — error log inspection console
- `debug-error-catcher.js` — standalone error capture

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | Main application — all HTML, CSS, and JavaScript (~4,060 lines) |
| `js/shared/contract.js` | Cross-tool localStorage contract with validators and safe accessors |
| `js/shared/ollama-client.js` | Ollama API client with streaming, typed errors, connection checks |
| `js/shared/prompts.js` | AI prompt templates for remediation, chat, and executive summaries |
| `debug.html` | Debug console — view error logs, check localStorage state |
| `debug-error-catcher.js` | Standalone error capture script |
| `404.html` | Hash routing fallback for direct URLs on GitHub Pages |
| `.nojekyll` | Disables Jekyll processing on GitHub Pages |
| `.github/workflows/pages.yml` | Auto-deploy to GitHub Pages on push to main |
| `README.md` | This file |

---

Built by David Chiang · [mrdavidchiang@gmail.com](mailto:mrdavidchiang@gmail.com)
