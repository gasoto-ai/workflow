# System Overview

This document describes the hardware, software, and agent setup running on George's Raspberry Pi 5.

---

## Hardware

**Primary machine:** Raspberry Pi 5 (8GB RAM, 4-core ARM Cortex-A76, ~117GB storage)
- Running 24/7 at home
- Connected via Tailscale VPN at `100.122.117.128`
- Connected to Discord via bot tokens for all three agents

**Secondary machine (available):** Raspberry Pi 3 Model B v1.2 (1GB RAM)
- Not currently in active use

---

## Software Stack

### OpenClaw
OpenClaw is the agent hosting platform. It runs on the Pi and manages:
- Agent sessions (Ren, Forge, Vex)
- Discord channel routing (inbound/outbound messages)
- Cron jobs for autonomous tasks
- Memory file management
- Tool access (web search, exec, file I/O)

Config: `/home/georges/.openclaw/openclaw.json`

### Agents

Three agents run on the Pi, each with their own workspace:

| Agent | Role | Workspace | Discord |
|---|---|---|---|
| **Ren** (main) | Planner, orchestrator, George's main interface | `/home/georges/.openclaw/workspace/` | Ren AI |
| **Forge** | Builder — code, PRs, TDD | `/home/georges/.openclaw/workspace-forge/` | Forge AI |
| **Vex** | QA gate — reviews PRs, hard pass/fail | `/home/georges/.openclaw/workspace-vex/` | Vex AI |

All agents communicate through the `#fish-tank` Discord channel.

### Channel
- **Discord guild:** Fish Tank
- **Channel:** `#fish-tank` (ID: `1481915397966921789`)
- George (Shrill) reads and participates here; Ren leads all conversations with George

---

## GitHub Setup

Two GitHub organizations:

| Org | Purpose | Access |
|---|---|---|
| `gasoto-ai` | Original org; portfolio + workflow live here | PAT in `/home/georges/.openclaw/workspace-forge/credentials.md` |
| `gasoto-dev` | Primary active org for all new projects | PAT in `/home/georges/.openclaw/workspace/credentials.md` |

**Note:** Both orgs are active. Migration to gasoto-dev as sole primary is in the ClickUp backlog.

### Active repos on gasoto-dev
- `checkout-demo` — Stripe e-commerce demo
- `webhook-inspector` — Live webhook debugger
- `agent-monitor` — Multi-agent activity dashboard
- `kelly-soto-photography` — Kelly Soto photography site
- `dashboard` — Team metrics dashboard (runs on Pi)
- `dungeon-cards` — Godot 4 roguelike deckbuilder
- `bomb-burritos` — Bomb Burritos LLC web site
- `bomb-burritos-app` — Bomb Burritos React Native app

### Active repos on gasoto-ai
- `portfolio` — George Soto's portfolio (live at codeyoursuccess.com)
- `workflow` — This repo; methodology + skill docs
- `the-crate`, `fish-tank-tools`, `codelens` — Earlier demo apps

---

## Memory System (L0/L1/L2)

Each agent maintains tiered memory:

**L0 — `memory/state.md`** (loaded every session)
Current project status, open tasks, active repos, auth notes. Both agents use this.

**L1 — `MEMORY.md`** (Ren only, main sessions with George)
Curated long-term memory: decisions, lessons, George context. Not loaded in group chats.

**L2 — `memory/projects/<repo>.md`** (loaded when working on that repo)
Per-repo context: stack, routes, DB schema, test setup, gotchas.

**Daily logs:** `memory/YYYY-MM-DD.md` — raw session notes, never deleted.

**Active task ownership:** `memory/active-tasks.md` — who owns what right now; prevents coordination conflicts.

---

## Autonomous Crons

| Schedule | Task |
|---|---|
| Daily 9am MST | Ren runs daily research |
| Monday 10am MST | Vex runs weekly repo audit |
| Friday afternoon MST | Ren posts weekly digest |

---

## Deployed Apps

| App | URL | Host |
|---|---|---|
| Portfolio | `codeyoursuccess.com` | Vercel |
| Dashboard | `http://100.122.117.128:3002` | Pi (Tailscale only) |

Other apps (checkout-demo, webhook-inspector, agent-monitor, bomb-burritos) are built but not yet deployed to Vercel.
