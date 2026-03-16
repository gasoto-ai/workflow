# MCP Transport Audit — 2026 Roadmap

> Research only. No config changes.
> Roadmap source: https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/ (March 9, 2026)
> Audited: 2026-03-16

---

## What the Roadmap Prioritizes

### 1. Transport Evolution & Scalability
The main change: MCP is evolving Streamable HTTP toward a **stateless session model** so servers can scale horizontally without holding session state. Also adding a `.well-known` metadata standard so server capabilities are discoverable without a live connection.

**What's NOT changing:** No new transports. The set stays small — just evolving Streamable HTTP. They explicitly said this is a deliberate decision.

### 2. Agent Communication
The Tasks primitive (SEP-1686) is getting lifecycle improvements: retry semantics for transient failures, expiry policies for result retention. This is iteration on an existing primitive, not a new pattern.

### 3. Governance Maturation
Contribution ladder + Working Group delegation. Not relevant to our stack.

### 4. Enterprise Readiness
Audit trails, SSO, gateway behavior. Not relevant to our stack.

---

## How We Currently Use MCP

### OpenClaw → Agent routing
OpenClaw routes messages between agents (Ren, Forge, Vex) via Discord channels. This is **OpenClaw's own internal routing** — not MCP protocol. Agent-to-agent communication goes through Discord messages with mention syntax (`<@USER_ID>`), not MCP sessions.

**MCP exposure: none.** OpenClaw handles session management internally. The roadmap transport changes don't apply here.

### chrome-devtools MCP (optional tool in qa/SKILL.md)
Our `qa/SKILL.md` references `chrome-devtools MCP` as an optional tool for browser-based validation. This uses the current MCP connection model — a local MCP server process that the agent connects to.

**Current behavior:** The skill explicitly says "if chrome-devtools MCP tools are NOT available, skip this phase." It's optional and never been active in our setup.

**Roadmap impact:** If we ever activate chrome-devtools MCP, the session model will evolve toward stateless Streamable HTTP. For a local tool (same machine as the agent), this is low risk — the transport changes are primarily targeted at remote/hosted MCP servers that need horizontal scaling. A local chrome-devtools MCP server running on the Pi doesn't need to scale horizontally.

### No other MCP connections found
Searched `openclaw.json`, all workspace skill files, and the OpenClaw config. No other MCP servers are configured or referenced.

---

## Gap Analysis

| Area | Current State | Roadmap Change | Impact |
|---|---|---|---|
| Agent-to-agent comms | Discord mentions via OpenClaw | Standardized agent comm patterns (SEP-1686 lifecycle) | **None** — we don't use MCP Tasks primitive; Discord is our transport |
| MCP session model | No active MCP sessions | Moving toward stateless Streamable HTTP | **None for now** — we have no stateful MCP sessions |
| chrome-devtools MCP | Optional, never activated | Session model will evolve | **Low** — local tool, no scaling concern |
| `.well-known` discovery | Not applicable | New capability discovery standard | **None** — we don't host MCP servers or use server registries |
| Enterprise features | N/A | Audit, SSO, gateway | **None** |

---

## Risk Assessment

**Overall: LOW**

We are MCP-light. Our stack is:
- OpenClaw (own routing layer, not MCP)
- Discord (agent comms transport)
- No hosted MCP servers
- No active MCP sessions
- chrome-devtools MCP is optional and unused

The 2026 roadmap transport evolution targets production deployments with remote MCP servers needing horizontal scaling. That's not us. The agent-to-agent standardization (Tasks lifecycle) is relevant in principle but doesn't affect our Discord-based comms.

---

## Recommended Next Steps

**No action required at this time.**

If we later:
1. **Add remote MCP servers** (e.g., a hosted tool server) — revisit the stateless transport spec before connecting
2. **Activate chrome-devtools MCP** — test with the current transport model; watch for breaking changes when the new spec ships
3. **Expand agent team significantly** — the Tasks primitive lifecycle improvements (retry, expiry) could become relevant if we adopt MCP-native agent coordination instead of Discord

**Watch list:** SEP-1686 (Tasks lifecycle) and the `.well-known` discovery spec — both worth following if we ever build MCP server infrastructure.

---

*Researched and written by Forge — 2026-03-16. Read-only audit. No config changes made.*
