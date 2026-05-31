---
name: paperclip-ai
description: "Interact with and manage a local Paperclip AI instance: inspect agents, create issues, communicate with agents, run doctor/status checks."
tags: [paperclip, multi-agent, orchestration, northpeak, zoe]
triggers:
  - user mentions Paperclip or the Paperclip setup
  - user wants to communicate with Zoe, Northpeak AI agents
  - user asks about the Paperclip company, agents, or issues
  - user wants to check Paperclip status or run diagnostics
---

# Paperclip AI — Local Instance Management

## Setup on Mac Mini

- **CLI:** `paperclipai` (npm global: `paperclipai@2026.428.0`)
- **Service:** LaunchAgent `com.paperclipai.server` (PID-tracked, auto-restarts)
- **Instance dir:** `~/.paperclip/instances/default/`
- **Port:** 3100 (bind: Tailscale)
- **Auth mode:** authenticated/private
- **Database:** Embedded PostgreSQL on port 54329

## Company: Northpeak AI

- **Company ID:** `e70adf27-b620-4fa5-b4c5-4d1297956c3f`
- **Shortname:** `NOR` (issue identifiers: NOR-xxx)

## Agent Fleet

| Name | Role | Reports To | Budget/mo |
|------|------|------------|-----------|
| **Zoe** | CEO | — | 100 € |
| **Titan** | CTO | Zoe | 100 € |
| **Aven** | General | Zoe | 20 € |
| **Aria** | CMO | Zoe | 15 € |
| **Nova** | Researcher | Zoe | 10 € |
| **Vera** | Researcher | Zoe | 10 € |
| **Atlas** | CFO | Aven | 10 € |
| **Hera** | General | Aven | 5 € |
| **Lexi** | General | Aven | 5 € |

Agent IDs: see `references/agent-ids.md`

## Key Commands

### Status & Health
```bash
# Service check
launchctl list | grep paperclip

# Doctor (all checks)
paperclipai doctor

# Dashboard summary
paperclipai dashboard get -C e70adf27-b620-4fa5-b4c5-4d1297956c3f --json

# List agents
paperclipai agent list -C e70adf27-b620-4fa5-b4c5-4d1297956c3f
```

### Communicating with Agents (via Issues)
Paperclip agents communicate via Issues — there is no direct chat API.

```bash
# Create an issue assigned to an agent
paperclipai issue create \
  -C e70adf27-b620-4fa5-b4c5-4d1297956c3f \
  --title "Your message title" \
  --description "Your message body" \
  --assignee-agent-id <agent-uuid> \
  --priority medium \
  --json

# Check issue status / read agent response
paperclipai issue get -C e70adf27-b620-4fa5-b4c5-4d1297956c3f <issueId>

# Add a follow-up comment
paperclipai issue comment -C e70adf27-b620-4fa5-b4c5-4d1297956c3f <issueId> --body "..."

# List open issues
paperclipai issue list -C e70adf27-b620-4fa5-b4c5-4d1297956c3f
```

### Activity & Approvals
```bash
paperclipai activity list -C e70adf27-b620-4fa5-b4c5-4d1297956c3f
paperclipai approval list -C e70adf27-b620-4fa5-b4c5-4d1297956c3f
```

## Communication Pattern

1. Create an Issue and assign it to the target agent
2. Issue lands in agent's backlog → agent picks it up (checkout)
3. Agent works the issue and marks it done / adds comments
4. Poll with `paperclipai issue get <id>` to read the response

## Architecture: Paperclip ↔ OpenClaw

**Critical:** The Paperclip agents are the SAME agents as the OpenClaw agents.

- **Paperclip** = Strategy & task management layer (Zoe as CEO)
- **OpenClaw** = Execution layer (same agents execute the tasks)
- **Zoe** = Paperclip-only (no OpenClaw equivalent)
- Flow: Luna/Kai → Paperclip Issue → Zoe → delegates → OpenClaw agent executes

| Paperclip Agent | OpenClaw ID | Role |
|----------------|-------------|------|
| Aven (COO) | main | Operations |
| Titan (CTO) | titan | Engineering |
| Atlas (CFO) | atlas | Finance |
| Vera (CRO) | vera | Research |
| Nova (CHLO) | nova | Health |
| Hera (CHO) | hera | Home/FeWo |
| Lexi (LEO) | lexi | Learning |
| Aria (CMO) | — | Marketing only |

**Delegation choice:**
- Quick/direct tasks → `openclaw agent --agent <id>` (cheaper, faster)
- Structured/tracked tasks → `paperclipai issue create` (Zoe routes it)

## Agent Memory Updates

OpenClaw agent memories are writable files — for bulk updates after interviews or profile changes, write directly to `~/.openclaw/workspace-<id>/MEMORY.md` instead of waking agents via LLM sessions. Cost-efficient and instant.

## Northpeak AI — Context & Goals

- **Domain:** northpeak-ai.com (registered; aitoolcheck.com was cancelled)
- **Goal:** Scalable passive income streams, 100% autonomous where possible
- **Company goal:** "Maximize Kai's professional productivity, automate personal operations, and build scalable AI-driven revenue streams."
- **Current projects:** Blog/SEO on northpeak-ai.com (NOR-47/48/49), affiliate marketing
- **Budget decisions:** Ad-hoc with Kai — no fixed monthly budget
- **Orchestration:** Vera/Nova/Hera (Haiku) + Lexi (Gemini Flash) for cheap research tasks; Zoe (Opus) only for strategic decisions

## Agent Memory — Bulk Updates

When interviewing Kai or collecting profile data, write **directly to MEMORY.md files** — do NOT wake agents via LLM sessions for bulk updates. Cost-efficient and instant:

```bash
# Direct file writes (no token cost)
~/.openclaw/workspace-hera/MEMORY.md
~/.openclaw/workspace-nova/MEMORY.md
~/.openclaw/workspace-atlas/MEMORY.md
~/.openclaw/workspace-titan/MEMORY.md
~/.openclaw/workspace-vera/MEMORY.md
~/.openclaw/workspace-lexi/MEMORY.md

# Shared profile (all agents read this)
~/.openclaw/workspace/shared/user_profile.md
```

## Paperclip Update

Check and apply updates when needed:

```bash
paperclipai --version          # installed version
npm show paperclipai version   # latest available

# If update available:
```bash
launchctl unload ~/Library/LaunchAgents/com.paperclipai.server.plist

# CORRECT — use the binary, not node directly:
# ProgramArguments must be: ["/opt/homebrew/bin/paperclipai", "run"]
# NOT: ["/opt/homebrew/bin/node", "/opt/homebrew/lib/node_modules/paperclipai/dist/index.js", "run"]

npm install -g paperclipai
rm -rf ~/.paperclip/instances/default/run 2>/dev/null
launchctl load ~/Library/LaunchAgents/com.paperclipai.server.plist
launchctl list | grep paperclip  # first column must be a PID, not "-"
```

Always check Paperclip version after an OpenClaw update — both services should be kept current together.

## Pitfalls

1. **No direct chat** — there is no `paperclipai chat` or `--message` flag. Issues are the only async communication channel.
2. **`-C` flag** works on `issue create` but NOT on `issue get` — use identifier directly: `paperclipai issue get NOR-104`
3. **Issue status starts as `backlog`** — agent must checkout before working. Use `paperclipai issue checkout <uuid> --agent-id <uuid>` to trigger immediately.
4. **`paperclipai -m`** does not exist — wrong mental model borrowed from OpenClaw.
5. **Polling for responses** — set a cron watcher rather than manually polling; agents are async and may take minutes to hours.
6. **Zoe runs on Opus** — expensive. Only assign strategic issues. Prefer direct OpenClaw calls for quick tasks.
7. **Activity feed reveals agent history** — `paperclipai activity list -C <company-id>` is useful for reconstructing what agents last worked on, their blockers, and open issues waiting for Kai's input.
8. **Vera's MEMORY.md may be empty** — was found empty in session 2026-05-24; recreated with full profile. Check before relying on it.
9. **Issues waiting for Kai** — check `in_review` issues regularly; agents like Zoe block on human input and won't progress until Kai responds (e.g. NOR-5 "Überlege Business Ideen" stalled since 03.05.2026).
10. **LaunchAgent MUST use the `paperclipai` binary, not `node dist/index.js` directly** — calling `node /opt/homebrew/lib/node_modules/paperclipai/dist/index.js run` triggers `import.meta.url === process.argv[1]` → `runCli()` (plugin-scaffolding code) fires instead of `main()`. Symptom: `Package package.json not found at /opt/homebrew/packages/shared/package.json`. Fix: set ProgramArguments to `["/opt/homebrew/bin/paperclipai", "run"]`. This was the root cause of *both* the 2026.517.0 and 2026.525.0 "failures" — they were not Paperclip bugs.
10. **LaunchAgent muss `paperclipai` Binary aufrufen, NICHT `node dist/index.js`** — wenn der LaunchAgent `node /opt/homebrew/lib/node_modules/paperclipai/dist/index.js run` aufruft, greift `import.meta.url === process.argv[1]` und ruft `runCli()` (Plugin-Scaffolding) statt `main()` auf → Fehler `Package package.json not found at /opt/homebrew/packages/shared/`. Fix: ProgramArguments auf `/opt/homebrew/bin/paperclipai run` setzen. **IMMER zuerst `paperclipai run` manuell testen bevor Fehler dem Hersteller zugeschrieben werden.**
11. **Diagnose vor Rollback:** Bei `Package package.json not found`-Fehler ZUERST `paperclipai run` manuell im Terminal testen. Wenn das funktioniert → Fehler liegt im LaunchAgent, nicht in der Software. Nicht sofort Rollback machen ohne Ursache zu kennen.
12. **Stale `/run` dir after failed start** — if service exits immediately, check logs for `Directory already exists: .../run`. Fix: `rm -rf ~/.paperclip/instances/default/run` then reload service.
13. **Service health check: PID column** — `launchctl list | grep paperclip` first column should be a PID number, NOT `-`. A `-` means the process exited. Always verify PID after service load.
