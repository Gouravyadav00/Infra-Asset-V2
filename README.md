# InfraKnit Version 2 — Roadmap & Upcoming Plan

**Document Version:** 2.0
**Date:** February 2026
**Classification:** Internal — Product Planning

---

## Table of Contents

1. [Current State (V1 Audit)](#1-current-state-v1-audit)
2. [Gap Analysis vs Market Leaders](#2-gap-analysis-vs-market-leaders)
3. [V2 Feature Roadmap](#3-v2-feature-roadmap)
4. [AI-Powered Features](#4-ai-powered-features)
5. [Server Specifications for 1,000 Assets](#5-server-specifications-for-1000-assets)
6. [Data Generation & Storage Estimates](#6-data-generation--storage-estimates)
7. [Architecture Changes for Scale](#7-architecture-changes-for-scale)
8. [Implementation Priority](#8-implementation-priority)

---

## 1. Current State (V1 Audit)

### What We Have (Fully Implemented)

| Area | Features |
|------|----------|
| **Agent Management** | Registration, heartbeat (60s), approval workflow, online/offline detection |
| **Asset Inventory** | Hardware (CPU, RAM, disk, BIOS), software (dpkg), OS info, network interfaces |
| **Peripheral Tracking** | Monitors, printers, USB devices, keyboards, mice (Windows only) |
| **Package Management** | Upload, catalog, deploy — Windows (.exe/.msi/.msu) + Linux (.deb with APT repo) |
| **Patch Management** | Windows (KB patches, .msu) + Linux (.deb), auto-discovery from Ubuntu mirrors |
| **Service Packs** | Bundle packages + patches, ordered deployment, stop-on-failure |
| **Job System** | Install, uninstall, patch, reboot, shutdown, run script, collect inventory |
| **Scheduling** | Once, daily, weekly, monthly, cron expressions |
| **Deployment Tracking** | Progress bars, success/failure counts, error logs per target |
| **Network Discovery** | ICMP ping + TCP fallback scanning, device registration |
| **Groups & Tags** | Static groups, dynamic groups, key-value tags |
| **Locations** | Hierarchical (region → building → floor → room → rack), map view with geocoding |
| **Notifications** | SMTP email + in-app notifications, alert rules |
| **Reporting** | CSV exports (agents, assets, jobs, packages, software), compliance reports |
| **User Management** | RBAC (admin, operator, viewer, auditor), JWT auth |
| **Audit Trail** | Full audit logging of admin actions |
| **Dashboard** | Real-time stats, charts, activity feed, system health |
| **Global Search** | Cross-entity search (assets, agents, jobs, packages, patches, groups, locations) |

### Platform Coverage

| Platform | Agent | Packages | Patches | Inventory |
|----------|-------|----------|---------|-----------|
| **Windows** | Planned | .exe, .msi, .ps1, .zip | .msu (KB) | Full |
| **Ubuntu Linux** | Done | .deb (APT repo) | .deb (auto-discovery) | Full |
| **macOS** | Not started | — | — | — |

### Known Agent Gaps (Ubuntu)

| Gap | Impact |
|-----|--------|
| Agent self-update not implemented | Can't patch the agent itself remotely |
| GPU detection missing | Incomplete hardware inventory for GPU workstations |
| Peripheral discovery empty on Linux | No monitor/printer/USB detection |
| TLS verification disabled by default | Security risk in production |
| Hardware grade hardcoded to "B" | No real scoring |

---

## 2. Gap Analysis vs Market Leaders

### Comparison: InfraKnit V1 vs Fleet vs ManageEngine Endpoint Central

| Feature | InfraKnit V1 | Fleet | ManageEngine |
|---------|:---:|:---:|:---:|
| Agent registration & heartbeat | Yes | Yes | Yes |
| Hardware inventory | Yes | Yes (osquery) | Yes |
| Software inventory | Yes | Yes (osquery) | Yes |
| OS information | Yes | Yes | Yes |
| Real-time metrics (CPU/RAM/disk) | Yes | Partial | Yes |
| Network interface tracking | Yes | Yes | Yes |
| Peripheral tracking | Partial | No | Yes |
| Package deployment (Windows) | Yes | No | Yes |
| Package deployment (Linux .deb) | Yes | No | Yes |
| Patch management (Windows KB) | Yes | No | Yes |
| Patch management (Linux auto-discovery) | Yes | No | Yes |
| **Third-party patch management** | **No** | No | **Yes** |
| Service packs (bundles) | Yes | No | Yes |
| Job scheduling (cron) | Yes | Partial | Yes |
| Deployment tracking | Yes | No | Yes |
| Network discovery | Yes | No | Yes |
| Groups & tags | Yes | Yes | Yes |
| Location hierarchy + map | Yes | No | Yes |
| RBAC (roles & permissions) | Yes | Yes | Yes |
| Audit trail | Yes | Yes | Yes |
| SMTP notifications | Yes | No | Yes |
| In-app notifications | Yes | No | Yes |
| CSV export & reports | Yes | Partial | Yes |
| Compliance reports | Basic | Yes (CIS) | **Yes (HIPAA, PCI, SOX, CIS)** |
| Global search | Yes | Yes (SQL) | Yes |
| **Vulnerability / CVE scanning** | **No** | **Yes** | **Yes** |
| **Remote desktop / terminal** | **No** | No | **Yes** |
| **Software license management** | **No** | No | **Yes** |
| **Application control (whitelist/blacklist)** | **No** | No | **Yes** |
| **USB device control (allow/block)** | **No** | No | **Yes** |
| **Encryption management (BitLocker/LUKS)** | **No** | **Yes** | **Yes** |
| **Configuration baselines & drift** | **No** | **Yes** (policies) | **Yes** |
| **Live query (SQL on endpoints)** | **No** | **Yes** (osquery) | No |
| **Bandwidth optimization (relay agents)** | **No** | No | **Yes** |
| **Power management** | **No** | No | **Yes** |
| **Browser security management** | **No** | No | **Yes** |
| **OS imaging / deployment** | **No** | No | **Yes** |
| **MDM (Mobile Device Management)** | **No** | **Yes** | **Yes** |
| **Webhook / SIEM integration** | **No** | **Yes** | **Yes** |
| **AI-powered analytics** | **No** | No | **Partial** |
| **Multi-tenant support** | **No** | **Yes** | **Yes** |
| **Agent self-update** | **No** | **Yes** | **Yes** |
| **Offline / air-gapped operation** | **Yes** | No | Partial |

### Our Unique Strengths

1. **Fully offline / air-gapped** — Agents install from on-prem APT repo, no internet needed
2. **Auto patch discovery** — Downloads Ubuntu indices, compares, builds local repo automatically
3. **Per-app APT isolation** — Each app/patch gets its own APT source, no repo conflicts
4. **Lightweight agent** — Minimal dependencies (psutil + requests only)
5. **Single-binary deployment** — No complex agent infrastructure

### Critical Gaps to Close for V2

The **bolded "No"** entries above are our gaps. Prioritized in Section 3.

---

## 3. V2 Feature Roadmap

### PHASE 1 — Security & Compliance (Weeks 1–6)

#### 1.1 Vulnerability / CVE Scanning
**What:** Match installed software versions against known CVE databases. Show which agents are vulnerable.

**How it works:**
```
1. Agent sends software inventory (already done — dpkg-query output)
2. Server syncs CVE database (NVD/OSV API or offline JSON feed)
3. Match: package_name + version → CVE IDs with severity scores (CVSS)
4. Dashboard: "47 agents have critical vulnerabilities"
5. One-click: Create patch jobs for affected packages
```

**Data source options:**
- Ubuntu Security Notices (USN) — free, Ubuntu-specific, XML feed
- OSV (Google Open Source Vulnerabilities) — free API, covers Linux packages
- NVD (NIST National Vulnerability Database) — free, comprehensive, JSON feeds
- For offline: download NVD JSON feed daily on a machine with internet, copy to server

**New components:**
- Model: `vulnerability` table (cve_id, package_name, affected_versions, severity, description)
- Service: `VulnerabilityService` — sync CVE data, match against inventory
- API: `/api/v1/vulnerabilities` — list, stats, by-agent, by-package
- Frontend: Vulnerability dashboard page, per-agent vulnerability tab

**Why it matters:** This is the #1 feature every enterprise buyer asks for. Fleet and ManageEngine both have it. Without it, InfraKnit is a deployment tool, not a security tool.

---

#### 1.2 Compliance Frameworks & Baselines
**What:** Pre-built compliance templates (CIS Benchmarks, custom baselines) that continuously check agent state.

**How it works:**
```
1. Admin creates a "baseline" (or uses a CIS template)
   Example: "All servers must have UFW enabled, OpenSSH ≥ 8.9, no root login"
2. Rules are evaluated against agent telemetry (snapshots, inventory, os_info)
3. Compliance score per agent: 85% (17/20 rules pass)
4. Non-compliant items highlighted with remediation steps
5. Compliance trend over time (was 60% last month, now 85%)
```

**Rule types:**
- Software installed/not installed (version constraints)
- Service running/stopped
- Firewall enabled
- Security settings (AppArmor, SELinux)
- OS version minimum
- Patch level (security patches applied within X days)
- Custom script check (run script, check exit code)

**New components:**
- Models: `compliance_baseline`, `compliance_rule`, `compliance_result`
- Service: `ComplianceService` — evaluate rules against agent state
- Background task: Re-evaluate compliance every hour
- Frontend: Compliance dashboard, baseline editor, per-agent compliance tab

---

#### 1.3 Agent Self-Update
**What:** Push new agent versions remotely without SSH access.

**How it works:**
```
1. Upload new agent package to server (like any other package)
2. Create "agent_update" job targeting agents
3. Agent downloads new version, verifies checksum
4. Runs update script: stop service → replace binary → start service
5. Reports back with new version number
```

Currently stubbed in `executor/engine.py:557` — needs actual implementation.

---

### PHASE 2 — Visibility & Control (Weeks 7–12)

#### 2.1 Remote Terminal (Web SSH)
**What:** Browser-based terminal to any managed Linux agent. No direct SSH needed.

**How it works:**
```
1. Admin clicks "Open Terminal" on agent detail page
2. Server establishes WebSocket connection to admin's browser
3. Server sends command to agent via existing job channel (or new WebSocket)
4. Agent executes command in PTY, streams output back
5. Admin sees real-time terminal in browser
```

**Implementation options:**
- **Option A (Simple):** Run-script jobs with real-time output streaming via WebSocket
- **Option B (Full):** WebSocket-based interactive PTY relay (like Teleport/MeshCentral)

**Recommendation:** Start with Option A (interactive script execution with live output). Much simpler, builds on existing job system. Add full PTY later.

---

#### 2.2 Application Control (Software Whitelist/Blacklist)
**What:** Define which software is allowed/blocked on managed endpoints.

**How it works:**
```
1. Admin creates policy: "Block: telegram, discord" or "Allow only: chrome, vscode, slack"
2. Agent checks installed software against policy (on inventory collection)
3. Violations reported to server
4. Optional: Auto-uninstall blocked software
5. Dashboard: "12 agents have unauthorized software"
```

**New components:**
- Model: `software_policy`, `software_policy_rule`, `policy_violation`
- Agent addition: Check inventory against policy, report violations
- Frontend: Policy editor, violation dashboard

---

#### 2.3 USB Device Control
**What:** Allow or block USB device types (storage, HID, etc.) per policy.

**How it works (Linux):**
```
1. Admin creates policy: "Block USB storage devices" or "Allow only keyboard + mouse"
2. Agent monitors udev events for USB connect/disconnect
3. Block: Create udev rule to disable unauthorized device types
4. Alert: Report to server when blocked device is connected
5. Audit: Full USB connection history
```

**New agent collector:** `usb_monitor.py` — watch udev events, report connect/disconnect.

---

#### 2.4 Encryption Management
**What:** Monitor and manage disk encryption (LUKS on Linux, BitLocker on Windows).

**How it works:**
```
1. Agent detects encryption status (lsblk + cryptsetup status)
2. Reports to server: encrypted/not encrypted, algorithm, key size
3. Dashboard: "34% of endpoints are not encrypted"
4. Optional: Push LUKS encryption via script job
```

**Agent addition:** Add to hardware collector — detect LUKS volumes.

---

### PHASE 3 — Scale & Integration (Weeks 13–18)

#### 3.1 Relay Agents (Distribution Points)
**What:** Agents that cache packages/patches locally and serve nearby agents. Reduces WAN bandwidth.

**Critical for 1000+ offline agents across multiple sites.**

```
Architecture:
                    ┌─────────────────┐
                    │  Master Server   │
                    │  (HQ - Windows)  │
                    └────────┬────────┘
                             │ WAN (or sneakernet)
               ┌─────────────┼─────────────┐
               │             │             │
        ┌──────┴──────┐ ┌───┴────┐ ┌──────┴──────┐
        │ Relay Agent  │ │ Relay  │ │ Relay Agent  │
        │ (Site A)     │ │(Site B)│ │ (Site C)     │
        └──────┬──────┘ └───┬────┘ └──────┬──────┘
               │             │             │
          ┌────┼────┐   ┌───┼───┐    ┌────┼────┐
          │    │    │   │   │   │    │    │    │
         Agent Agent   Agent Agent  Agent Agent
         (50)  (50)    (30) (30)   (40)  (40)
```

**How it works:**
1. Relay agent syncs apt-catalog from master server
2. Nearby agents configured to pull from relay instead of master
3. Job polling still goes to master (small payload)
4. Package/patch downloads go to relay (large payload)
5. Relay keeps local cache, only downloads new versions from master

---

#### 3.2 Webhook & SIEM Integration
**What:** Push events to external systems (Slack, Teams, Splunk, ELK, Wazuh).

**Event types:**
- Agent came online / went offline
- Job completed / failed
- Vulnerability detected (critical)
- Compliance violation
- Unauthorized software detected
- USB device blocked

**Implementation:**
- Model: `webhook_config` (url, events, headers, secret)
- Service: `WebhookService` — HTTP POST with HMAC signature
- Frontend: Webhook configuration page

---

#### 3.3 Software License Management
**What:** Track license usage vs purchased count. Alert on over-deployment.

```
1. Admin enters: "Adobe Acrobat — 50 licenses purchased"
2. Software inventory shows: Installed on 67 agents
3. Alert: "17 agents over license limit"
4. Report: License compliance summary for audit
```

**New components:**
- Model: `software_license` (software_name, vendor, license_count, license_type, expiry_date)
- Service: Match installed software against license records
- Frontend: License management page, compliance dashboard

---

#### 3.4 Configuration Baselines & Drift Detection
**What:** Define expected system configuration, detect when agents drift from it.

```
1. Admin defines baseline: "sshd_config must have PermitRootLogin=no"
2. Agent collects config file checksums / key settings
3. Server compares against baseline
4. Drift detected: "Agent X changed sshd_config — PermitRootLogin=yes"
5. Optional: Auto-remediate by pushing correct config
```

---

#### 3.5 Windows Patch Auto-Discovery
**What:** Same as Linux patch discovery, but for Windows.

**How it works:**
- Download Microsoft Update Catalog data (WSUS offline scan cab)
- Or use Windows Update Agent API on endpoints
- Compare with installed KBs from agent inventory
- Show available patches, download .msu, create patch records

---

#### 3.6 Third-Party Patch Management
**What:** Auto-discover and deploy updates for non-OS software (Chrome, Firefox, 7-Zip, VLC, etc.)

**Data sources:**
- Chocolatey/winget package metadata (Windows)
- Snap/Flatpak metadata (Linux)
- Custom catalog of common software versions + download URLs
- Vendor RSS feeds (Google Chrome releases, Mozilla releases)

---

### PHASE 4 — Enterprise & Polish (Weeks 19–24)

#### 4.1 Multi-Tenant Support
**What:** Single server manages multiple organizations with data isolation.

- Tenant-level data isolation (all queries filtered by tenant_id)
- Tenant admin vs super admin roles
- Per-tenant branding and settings
- Useful for MSPs managing multiple clients

#### 4.2 Power Management
**What:** Schedule wake/sleep/hibernate. Wake-on-LAN for patching windows.

- Agent reports power state and wake capability
- Server sends Wake-on-LAN packets (or via relay agents on same subnet)
- Schedule: "Wake all agents at 2 AM, patch, shutdown at 4 AM"

#### 4.3 Custom Dashboards
**What:** Admin builds their own dashboard with drag-and-drop widgets.

- Widget types: stat card, chart, table, map, compliance gauge
- Save/load dashboard layouts per user
- Share dashboards with other users

#### 4.4 Backup & Disaster Recovery
**What:** Automated server backup + restore procedure.

- Database backup (mysqldump / sqlite export) on schedule
- APT catalog backup (tar archive)
- Configuration backup
- One-click restore
- Export/import server state for migration

#### 4.5 Advanced Reporting
**What:** Scheduled PDF/Excel reports emailed to stakeholders.

- Report templates: Compliance summary, patch status, vulnerability report, asset inventory
- Schedule: Daily/weekly/monthly
- Email distribution lists
- PDF generation with charts and tables

---

## 4. AI-Powered Features

### 4.1 Anomaly Detection (High Impact)
**What:** Detect unusual patterns in agent metrics automatically.

```
How it works:
1. Collect baseline: Learn normal CPU/RAM/disk patterns per agent over 2 weeks
2. Real-time: Compare incoming metrics against learned baseline
3. Alert: "Agent X CPU spiked to 95% — unusual for this time of day"
4. Context: Show what changed (new process? failed service? crypto miner?)
```

**Model:** Time-series anomaly detection (Isolation Forest or simple statistical — Z-score with rolling window). No GPU needed — runs on CPU.

**Value:** Detect compromised machines (crypto miners), failing hardware (disk I/O anomalies), misconfigured services (memory leaks) before they cause outages.

---

### 4.2 Smart Patch Prioritization (High Impact)
**What:** AI ranks patches by actual risk to YOUR environment, not just CVSS score.

```
Factors:
- CVSS severity (from CVE database)
- Is the vulnerable service exposed to network? (from snapshot — listening ports)
- How many agents affected? (agent_count from update_catalog)
- Is firewall enabled? (from security status)
- Is it internet-facing? (from network config / location)
- Historical exploit activity (from threat feeds)

Output:
- Risk score: 0–100 per patch
- "Patch openssl FIRST — 47 agents, network-exposed, CVSS 9.8, exploit in wild"
- "libpng can wait — 12 agents, no network exposure, CVSS 5.3, no known exploit"
```

**Value:** Admins patch what matters most first instead of patching everything blindly.

---

### 4.3 Natural Language Query (Medium Impact)
**What:** Ask questions in English instead of building filters.

```
Admin types: "Show me all servers in Mumbai running OpenSSL older than 3.0"
System translates to: SELECT agents WHERE location LIKE 'Mumbai%' AND software
                       LIKE 'openssl' AND version < '3.0' AND asset_type = 'server'
Admin types: "Which agents haven't been patched in 30 days?"
System translates to: Agents WHERE last_patch_job > 30 days ago
```

**Implementation:** Use Claude API (or local LLM) to convert natural language → SQL/API filters. The LLM gets the database schema as context and generates safe, read-only queries.

**Value:** Non-technical managers can get answers without learning the UI.

---

### 4.4 Predictive Maintenance (Medium Impact)
**What:** Predict hardware failures before they happen.

```
Disk failure prediction:
- Track disk read/write error rates over time (from SMART data)
- Track disk usage growth rate
- Alert: "Server X disk will be full in 12 days at current growth rate"
- Alert: "Server Y disk showing increasing SMART errors — replace within 30 days"

Memory degradation:
- Track ECC error counts (if available)
- Alert: "Server Z showing increasing memory errors — DIMM 2 may be failing"
```

**Agent addition:** Collect SMART data from drives (smartctl).

---

### 4.5 Auto-Remediation Suggestions (Medium Impact)
**What:** When a job fails, AI suggests what to fix.

```
Job failed: "apt-get install nginx" → "E: Unable to locate package nginx"

AI analysis:
- APT source for nginx is not configured
- Similar successful installs on other agents used per-app source
- Suggestion: "The APT source file is missing. Re-trigger the job or
  manually add the source: deb [trusted=yes] http://server/apt-catalog/..."

Job failed: "dpkg: error processing package (dependency problems)"

AI analysis:
- Package X depends on libfoo >= 2.0, but libfoo 1.8 is installed
- libfoo 2.0 is available in update_catalog but not yet downloaded
- Suggestion: "Download and deploy libfoo 2.0 first, then retry this package"
```

**Implementation:** Feed error logs + context to Claude API. Return actionable suggestion.

---

### 4.6 Intelligent Asset Classification (Low Impact)
**What:** Auto-tag and categorize assets based on their software/hardware profile.

```
Agent has: nginx, postgresql, redis, docker → Tag: "web-server", "database-server"
Agent has: libreoffice, thunderbird, firefox → Tag: "desktop-workstation"
Agent has: nvidia-driver, cuda, tensorflow → Tag: "ml-workstation", "gpu-compute"
Agent has: samba, nfs-kernel-server → Tag: "file-server"
```

**Implementation:** Rule-based first (fast, no API calls). AI-enhanced later for edge cases.

**Value:** Auto-populate dynamic groups. New agent is auto-classified within minutes.

---

### 4.7 Compliance Gap Analysis (Low Impact)
**What:** AI reads compliance framework requirements and maps them to existing telemetry.

```
CIS Ubuntu 22.04 Benchmark, Rule 5.2.4: "Ensure SSH PermitRootLogin is disabled"

AI maps to: Check snapshot → services → sshd config → PermitRootLogin
Current data: We collect service status but NOT sshd_config values
Gap: "Need to add sshd_config collection to agent snapshot"

Output: Coverage report — "InfraKnit covers 67/100 CIS rules.
         23 rules need new collectors. 10 rules are not applicable."
```

---

### AI Implementation Summary

| Feature | Complexity | AI Type | API Cost | Offline Capable |
|---------|-----------|---------|----------|-----------------|
| Anomaly Detection | Medium | Statistical (no LLM) | Free | Yes |
| Smart Patch Priority | Medium | Scoring algorithm + CVE data | Free | Yes |
| Natural Language Query | Medium | LLM (Claude API) | ~$0.01/query | No (needs API) |
| Predictive Maintenance | Low | Statistical (trend analysis) | Free | Yes |
| Auto-Remediation | Low | LLM (Claude API) | ~$0.02/suggestion | No (needs API) |
| Asset Classification | Low | Rule-based + optional LLM | Free / ~$0.005 | Yes (rule-based) |
| Compliance Gap Analysis | High | LLM + custom rules | ~$0.05/analysis | No (needs API) |

**Note:** Anomaly detection, patch prioritization, and predictive maintenance are pure algorithms — no LLM API needed. They run entirely on-server and work offline.

---

## 5. Server Specifications for 1,000 Assets

### Deployment Scenario

```
- 1,000 Ubuntu agents (mix of jammy + noble)
- 100% offline (air-gapped LAN)
- All agents report to single server
- Heartbeat: every 60 seconds
- Metrics: every 60 seconds
- Snapshots: every 5 minutes
- Inventory: every 24 hours
- Hardware: every 24 hours
- Job polling: every 30 seconds
```

### Request Load Calculation

| Request Type | Interval | Requests/min (1000 agents) | Payload Size |
|-------------|----------|---------------------------|--------------|
| Heartbeat | 60s | 1,000 | ~0.5 KB |
| Metrics ingest | 60s | 1,000 | ~0.8 KB |
| Snapshot ingest | 300s | 200 | ~15 KB |
| Inventory ingest | 86400s | ~1 | ~500 KB |
| Hardware ingest | 86400s | ~1 | ~5 KB |
| OS info ingest | 86400s | ~1 | ~2 KB |
| Job poll | 30s | 2,000 | ~0.2 KB |
| **Total** | | **~4,200 req/min** | |
|  | | **~70 req/sec** | |

### Server Hardware Requirements

#### Minimum (1,000 agents, basic operation)

| Component | Specification | Rationale |
|-----------|--------------|-----------|
| **CPU** | 4 cores / 8 threads (Intel i5-12400 or Xeon E-2300 series) | FastAPI with uvicorn workers handles ~200 req/sec on 4 cores |
| **RAM** | 16 GB DDR4 | MySQL buffer pool (4 GB) + Python workers (8 GB) + OS (4 GB) |
| **Storage** | 500 GB SSD (NVMe preferred) | Database (~50 GB) + APT catalog (~100 GB) + OS + headroom |
| **Network** | 1 Gbps Ethernet | Sufficient for 1000 agents on LAN |
| **OS** | Windows Server 2022 Standard | Current deployment target |

#### Recommended (1,000 agents, all V2 features, growth headroom)

| Component | Specification | Rationale |
|-----------|--------------|-----------|
| **CPU** | 8 cores / 16 threads (Intel Xeon E-2400 or AMD EPYC 4004) | CVE scanning, compliance checks, background sync all CPU-bound |
| **RAM** | 32 GB DDR5 | MySQL buffer pool (8 GB) + Python workers (16 GB) + cache (4 GB) + OS (4 GB) |
| **System Disk** | 256 GB NVMe SSD | OS + application |
| **Data Disk** | 1 TB NVMe SSD | Database + APT catalog + patch storage + backups |
| **Network** | 1 Gbps Ethernet (2x for redundancy) | Sufficient for 2,000+ agents |
| **OS** | Windows Server 2022 Standard | Current deployment target |
| **UPS** | 1000 VA | Prevent database corruption on power loss |

#### Future-Proof (2,500+ agents, multi-site with relays)

| Component | Specification |
|-----------|--------------|
| **CPU** | 16 cores (Xeon W or dual socket) |
| **RAM** | 64 GB DDR5 |
| **Storage** | 2 TB NVMe RAID-1 (mirrored) |
| **Network** | 10 Gbps |
| **Database** | Dedicated MySQL server (separate machine) |

### Software Stack

| Component | Current | Recommended for 1,000 |
|-----------|---------|----------------------|
| **Python** | 3.11+ | Same |
| **FastAPI** | Latest | Same + `--workers 4` (match CPU cores) |
| **Database** | MySQL 8.0 at 10.10.11.80 | MySQL 8.0 with tuned my.cnf (below) |
| **Reverse Proxy** | None | Nginx or Caddy (TLS termination, static file serving) |
| **Process Manager** | Manual uvicorn | systemd service or NSSM (Windows) |

### MySQL Tuning (my.cnf) for 1,000 Agents

```ini
[mysqld]
# Buffer pool — 25% of RAM
innodb_buffer_pool_size = 4G

# Connection handling
max_connections = 200
wait_timeout = 300
interactive_timeout = 300

# Write performance
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2    # Trade durability for speed (OK with UPS)
innodb_flush_method = O_DIRECT

# Query cache (disabled in MySQL 8, use ProxySQL if needed)
# Read performance
innodb_read_io_threads = 8
innodb_write_io_threads = 8

# Slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

---

## 6. Data Generation & Storage Estimates

### Per-Agent Daily Data Generation

| Data Type | Frequency | Records/Day | Size/Record | Daily Total |
|-----------|----------|-------------|-------------|-------------|
| Heartbeat | 60s | 1,440 | 0.5 KB | 0.7 MB |
| Metrics | 60s | 1,440 | 0.8 KB | 1.1 MB |
| Snapshots | 300s | 288 | 15 KB | 4.2 MB |
| Software Inventory | 24h | 1 (500 rows) | 500 KB | 0.5 MB |
| Hardware Inventory | 24h | 1 | 5 KB | 0.005 MB |
| OS Info | 24h | 1 | 2 KB | 0.002 MB |
| Job Poll | 30s | 2,880 | 0.2 KB | 0.6 MB |
| **Per-Agent Total** | | | | **~7.1 MB/day** |

### 1,000 Agents — Raw Data Generation

| Period | Raw Data Volume |
|--------|----------------|
| **Per Day** | 7.1 GB |
| **Per Week** | 49.7 GB |
| **Per Month** | 213 GB |
| **Per Year** | **2.6 TB (raw)** |

### With Retention Policies (Actual Storage)

InfraKnit has built-in retention policies. Old data is purged automatically.

| Data Type | Retention | Storage (1,000 agents) |
|-----------|----------|----------------------|
| Metrics | 30 days | 33 GB |
| Snapshots | 7 days | 29.4 GB |
| Software Inventory | Latest only (overwritten) | 0.5 GB |
| Hardware Inventory | Latest only (overwritten) | 0.005 GB |
| OS Info | Latest only (overwritten) | 0.002 GB |
| Heartbeats | Latest only (agent row updated) | Negligible |
| Job History | 90 days | ~2 GB (depends on job volume) |
| Audit Logs | 365 days | ~1 GB |
| **Subtotal (DB)** | | **~66 GB** |

### APT Catalog Storage (Patches & Packages)

| Item | Estimate |
|------|----------|
| Average .deb file size | 2 MB |
| Patches per codename | ~200 (security + updates) |
| Codenames | 3 (focal, jammy, noble) |
| Packages deployed | ~50 |
| **APT catalog total** | ~200 × 3 × 2 MB + 50 × 2 MB = **~1.3 GB** |
| Growth per year | +500 MB (new patches) |

### Total Storage Forecast

| Component | Year 1 | Year 2 | Year 3 |
|-----------|--------|--------|--------|
| Database (with retention) | 66 GB | 70 GB | 75 GB |
| APT Catalog | 1.3 GB | 1.8 GB | 2.3 GB |
| Backups (3 copies) | 200 GB | 215 GB | 230 GB |
| Database growth (no purge on jobs/audit) | +10 GB | +20 GB | +30 GB |
| **Total** | **~280 GB** | **~310 GB** | **~340 GB** |

**Conclusion:** A 500 GB SSD is sufficient for Year 1. A 1 TB SSD provides 3-year headroom.

### Network Bandwidth

| Direction | Calculation | Bandwidth |
|-----------|------------|-----------|
| Agents → Server (telemetry) | 1000 × 7.1 MB/day = 7.1 GB/day | ~0.7 Mbps sustained |
| Server → Agents (job poll responses) | 1000 × 0.6 MB/day | ~0.06 Mbps |
| Server → Agents (patch downloads, burst) | 1000 × 2 MB/patch × 10 patches | 20 GB burst |
| **Steady state** | | **< 1 Mbps** |
| **Patch deployment burst** | 1000 agents × 2 MB | **Up to 100 Mbps for 30 min** |

**Conclusion:** 1 Gbps is more than sufficient. Patch deployment is the only burst scenario.

---

## 7. Architecture Changes for Scale

### Current Architecture (V1)

```
┌──────────┐     HTTP     ┌────────────────────┐
│  Agent   │ ───────────→ │  FastAPI (uvicorn)  │
│  ×1000   │ ←─────────── │  Single process     │
└──────────┘              │  SQLite/MySQL       │
                          │  File storage       │
                          └────────────────────┘
```

### Recommended Architecture (V2, 1,000 agents)

```
┌──────────┐         ┌───────────────────────────────────┐
│  Agent   │  HTTPS  │         InfraKnit Server          │
│  ×1000   │ ──────→ │                                   │
└──────────┘         │  ┌─────────┐    ┌──────────────┐  │
                     │  │ Nginx   │───→│ FastAPI       │  │
                     │  │ (TLS +  │    │ (4 workers)   │  │
                     │  │  static)│    └──────┬───────┘  │
                     │  └─────────┘           │          │
                     │                   ┌────┴────┐     │
                     │                   │  MySQL  │     │
                     │                   │  (local │     │
                     │                   │  or     │     │
                     │                   │  remote)│     │
                     │                   └─────────┘     │
                     │                                   │
                     │  ┌─────────────────────────────┐  │
                     │  │ apt-catalog/ (served by     │  │
                     │  │ Nginx as static files)      │  │
                     │  └─────────────────────────────┘  │
                     └───────────────────────────────────┘
```

### Key Changes for Scale

| Change | Why |
|--------|-----|
| **Nginx reverse proxy** | TLS termination, static file serving for apt-catalog (faster than Python), connection buffering |
| **4 uvicorn workers** | Utilize all CPU cores. Each worker handles ~50 req/sec |
| **MySQL on SSD** | Required for query performance at this scale |
| **Database indexes** | Add composite indexes on hot query paths (agent_id + timestamp) |
| **Connection pooling** | Already configured (pool_size=10, max_overflow=20). Sufficient for 4 workers |
| **Metrics aggregation** | Store hourly averages instead of per-minute after 7 days |
| **Background task queue** | Replace threading.Thread with a proper task queue (Celery + Redis or simpler: APScheduler) for CVE sync, patch downloads |

### Multi-Site Architecture (2,500+ agents)

```
                    ┌──────────────────┐
                    │  Master Server   │
                    │  (HQ)            │
                    │  MySQL + FastAPI  │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │ WAN/VPN      │              │ Sneakernet
              │              │              │ (USB drive)
       ┌──────┴──────┐ ┌────┴─────┐ ┌──────┴──────┐
       │ Relay Agent  │ │  Relay   │ │ Relay Agent  │
       │ (Site A)     │ │ (Site B) │ │ (Air-gapped) │
       │ apt-catalog  │ │ apt-cat  │ │ apt-catalog   │
       │ mirror       │ │ mirror   │ │ (manual sync) │
       └──────┬──────┘ └────┬─────┘ └──────┬──────┘
              │              │              │
         500 agents     500 agents    1500 agents
```

---

## 8. Implementation Priority

### Priority Matrix

| # | Feature | Business Value | Effort | Priority |
|---|---------|:---:|:---:|:---:|
| 1 | Vulnerability / CVE Scanning | Critical | 3 weeks | P0 |
| 2 | Agent Self-Update | Critical | 1 week | P0 |
| 3 | Compliance Baselines | High | 3 weeks | P1 |
| 4 | Remote Terminal (web SSH) | High | 2 weeks | P1 |
| 5 | Anomaly Detection (AI) | High | 2 weeks | P1 |
| 6 | Smart Patch Prioritization (AI) | High | 1 week | P1 |
| 7 | Application Control | Medium | 2 weeks | P2 |
| 8 | USB Device Control | Medium | 2 weeks | P2 |
| 9 | Encryption Management | Medium | 1 week | P2 |
| 10 | Webhook / SIEM Integration | Medium | 1 week | P2 |
| 11 | Relay Agents | High (for scale) | 4 weeks | P2 |
| 12 | Software License Management | Medium | 2 weeks | P2 |
| 13 | Configuration Drift Detection | Medium | 3 weeks | P3 |
| 14 | Windows Patch Auto-Discovery | High | 3 weeks | P3 |
| 15 | Third-Party Patch Management | Medium | 4 weeks | P3 |
| 16 | Natural Language Query (AI) | Low | 2 weeks | P3 |
| 17 | Multi-Tenant Support | Low (unless MSP) | 4 weeks | P4 |
| 18 | Power Management | Low | 1 week | P4 |
| 19 | Custom Dashboards | Low | 3 weeks | P4 |
| 20 | Advanced Reporting (PDF) | Medium | 2 weeks | P4 |
| 21 | Auto-Remediation (AI) | Low | 1 week | P4 |
| 22 | Predictive Maintenance (AI) | Low | 2 weeks | P4 |
| 23 | Asset Classification (AI) | Low | 1 week | P4 |

### Recommended Delivery Timeline

```
Month 1-2:  CVE Scanning + Agent Self-Update + Compliance Baselines
            → InfraKnit becomes a security tool, not just deployment

Month 3-4:  Remote Terminal + Anomaly Detection + Smart Patch Priority
            → AI differentiation, operational efficiency

Month 5-6:  App Control + USB Control + Encryption + Webhooks
            → Enterprise security posture management

Month 7-9:  Relay Agents + License Mgmt + Config Drift + Windows Patch Discovery
            → Scale to 2,500+ agents, complete feature parity

Month 10-12: Multi-tenant + Custom Dashboards + Advanced Reports + NLP Query
             → Enterprise polish, MSP readiness
```

---

## Summary

InfraKnit V1 is a **solid, feature-complete** endpoint management platform with 150+ API endpoints, 24 frontend pages, and full Linux agent coverage. The unique strength — **fully offline operation with auto-patching** — is unmatched by Fleet or ManageEngine.

V2 closes the gap on **security visibility** (CVE scanning, compliance), **operational control** (remote terminal, app control, USB control), and **intelligence** (AI-powered anomaly detection and patch prioritization) — while maintaining the offline-first architecture that makes InfraKnit unique.

For 1,000 agents: an 8-core, 32 GB RAM server with 1 TB SSD handles everything comfortably. Annual data storage with retention policies is ~280 GB. Network bandwidth is negligible (< 1 Mbps sustained).

---

*InfraKnit — Manage everything. Trust nothing to the cloud.*
