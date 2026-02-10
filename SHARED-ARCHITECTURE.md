# Shared Architecture: Claude Code + OpenClaw

**Date:** 2026-02-09
**Purpose:** How to connect Claude Code (Max plan) and OpenClaw to share memory, hand off tasks, and create agents that DO things

---

## Current State of Mark's OpenClaw Workspace

| File | Status | Purpose |
|------|--------|---------|
| `AGENTS.md` | Well-developed | Memory patterns, heartbeat system, group chat rules, action words |
| `SOUL.md` | Good + hardened | Personality, boundaries, security rules (added 2026-02-09) |
| `Working-With-OpenClaw-Guide.md` | Comprehensive | Agent self-documentation of capabilities and patterns |
| `IDENTITY.md` | Unfilled template | Agent hasn't personalized yet |
| `USER.md` | Unfilled template | No user profile populated |
| `TOOLS.md` | Template only | No environment-specific notes yet |
| `HEARTBEAT.md` | Empty | No proactive tasks configured |
| `memory/` | 2 session logs | Sparse — needs more use to build up |
| `references/` | 1 file (Antfarm) | Tweet summary of multi-agent workflows |

**Location:** `~/.openclaw/workspace/`

---

## Is OpenClaw the Right Tool for Task-Doing Agents?

### Yes. Here's why:

OpenClaw provides the **infrastructure layer** that Claude Code lacks:

| Capability | OpenClaw | Claude Code | Building It Yourself |
|------------|----------|-------------|---------------------|
| Always-on daemon | Built-in (LaunchAgent) | Not possible | Weeks of work |
| Telegram messaging | Built-in, polling mode | Not possible | Days (basic bot) |
| Browser CDP control | Managed Chrome on port 18800 | Not possible | Significant effort |
| Heartbeat/proactive checks | Built-in (HEARTBEAT.md) | Not possible | Custom cron scripts |
| Cron scheduled tasks | Built-in | Not possible | Basic (just cron) |
| Sub-agents | Built-in (background sessions) | Not possible | Complex |
| Session persistence | Compaction + session logs | Lost on exit | Custom persistence layer |
| Multi-channel | Telegram, WhatsApp, Discord, etc. | Terminal only | Months per channel |
| Skills platform | Installable skills marketplace | Slash commands only | Custom plugin system |
| Web search | Brave API integration | Built-in (different) | API integration |

### The alternatives and why they're worse:

1. **Build it yourself on Claude Code** — You'd need to build a daemon, Telegram bot, browser CDP integration, session management, memory system, heartbeat/cron, tool execution framework. Months of work to replicate what OpenClaw already does.

2. **Other frameworks (AutoGPT, CrewAI, LangGraph)** — Framework-level tools, not ready-to-run assistants. No Telegram integration, no managed browser, no personal assistant architecture. Still need to build the infrastructure layer.

3. **Antfarm on OpenClaw** — Multi-agent workflows built ON TOP of OpenClaw. Uses the "Ralph loop" pattern (fresh context per session, memory through git). Directly relevant to Mark's goals. See `references/antfarm-openclaw-agent-teams.md`.

---

## How the Two Systems Connect

### The Filesystem IS the Shared Memory

Both Claude Code and OpenClaw can read and write to the same Mac filesystem. No special integration needed — just shared conventions about where things live.

**Claude Code can access:**
- `~/.openclaw/workspace/` — all agent files, memory, references
- `~/.openclaw/openclaw.json` — config
- `~/.openclaw/agents/main/sessions/*.jsonl` — full conversation history
- Everything else on the Mac

**OpenClaw can access:**
- Its workspace (`~/.openclaw/workspace/`)
- Browser (CDP on port 18800)
- Web search (Brave API)
- Shell commands (exec, with approval)
- Any file on the Mac (via exec/read tools)

### Shared Resource Map

```
~/.openclaw/workspace/              ← Both systems read/write here
├── AGENTS.md                       ← Agent behavior rules
├── SOUL.md                         ← Agent personality + security
├── IDENTITY.md                     ← Agent identity
├── USER.md                         ← User profile
├── TOOLS.md                        ← Environment-specific notes
├── HEARTBEAT.md                    ← Proactive task list
├── memory/                         ← Agent session memory
│   ├── YYYY-MM-DD.md              ← Daily logs
│   └── MEMORY.md                  ← (future) curated long-term memory
├── references/                     ← Saved research/summaries
├── tasks/                          ← (future) task handoff queue
└── Working-With-OpenClaw-Guide.md  ← Agent self-docs

~/PycharmProjects/openclaw/         ← Claude Code project directory
├── BACKSTORY.md                    ← Session history (LAWS system)
├── CLAUDE-CODE-OPENCLAW-STRATEGY.md
├── SHARED-ARCHITECTURE.md          ← This file
└── OPENCLAW_SECURITY_HARDENING.md

~/.claude/projects/.../memory/      ← Claude Code auto-memory
└── MEMORY.md                       ← CC's persistent notes about this project
```

---

## Task Handoff Patterns

### Pattern 1: Telegram → OpenClaw → Claude Code

**Use when:** You get an idea on the go, need Claude Code's power later.

```
1. You message OpenClaw via Telegram:
   "Research X and save findings to references/X.md"

2. OpenClaw uses browser/web_search, writes:
   ~/.openclaw/workspace/references/X.md

3. Later, you SSH in and open Claude Code:
   claude
   > "Read ~/.openclaw/workspace/references/X.md and build this"

4. Claude Code does the heavy coding/analysis

5. Claude Code writes results back to workspace if needed
```

### Pattern 2: Claude Code → OpenClaw

**Use when:** Claude Code produces something the agent should know about.

```
1. Claude Code writes a task or reference file:
   ~/.openclaw/workspace/tasks/implement-feature.md

2. You tell OpenClaw via Telegram:
   "Read tasks/implement-feature.md and execute it"

3. OpenClaw picks it up and acts on it
```

### Pattern 3: Shared Context via BACKSTORY.md

**Use when:** Both systems need to understand project history.

```
1. Claude Code updates BACKSTORY.md via /savenow skill
2. OpenClaw can read it: "Read ~/PycharmProjects/openclaw/BACKSTORY.md"
3. Both systems have the same project context
```

### Pattern 4: Heartbeat-Driven Background Work

**Use when:** You want OpenClaw to check things periodically without being asked.

```
1. Add tasks to HEARTBEAT.md:
   - Check for new GitHub issues on emmalone/openclaw
   - Monitor gateway health
   - Review and organize memory files

2. OpenClaw checks these on its heartbeat interval

3. If something needs attention, it messages you on Telegram

4. You can then SSH in and use Claude Code if heavy work is needed
```

---

## What OpenClaw Does Best (Keep Using It For)

1. **Real-time Telegram messaging** — Quick questions, forwarding URLs, getting summaries
2. **Browser automation** — Reading tweets, fetching pages that block server-side requests
3. **Proactive monitoring** — Heartbeat checks, cron jobs, scheduled reminders
4. **Always-on availability** — Messages get answered even when you're not in a terminal
5. **Web research** — Brave Search + browser CDP for comprehensive web access
6. **File-based task execution** — "Read this spec and do it"

## What Claude Code Does Best (Keep Using It For)

1. **All coding and development** — Best AI for code, free on Max plan
2. **Complex multi-file analysis** — Plan mode, codebase understanding
3. **System administration** — Full shell access, git integration
4. **Heavy research and synthesis** — Unlimited context, no cost pressure
5. **Architecture and planning** — EnterPlanMode, task lists, structured thinking
6. **This kind of strategic analysis** — Long-form thinking without token cost concerns

---

## Antfarm: Worth Exploring

The Antfarm reference your agent saved (`references/antfarm-openclaw-agent-teams.md`) describes exactly the multi-agent workflow system you're talking about:

- **Deterministic workflows** — Same steps, same order, every time
- **Agents verify each other** — Developer doesn't mark their own homework
- **Fresh context per session** — Ralph loop pattern, no context bloat
- **Zero infrastructure** — YAML + SQLite + cron, no Docker/Redis/Kafka
- **Built for OpenClaw** — One-command install

**Bundled workflows:**
- `feature-dev` (7 agents) — Feature request → tested PR
- `security-audit` (7 agents) — Repo → security fix PR
- `bug-fix` (6 agents) — Bug report → fix with regression test

**Repo:** https://github.com/snarktank/antfarm

This could be the bridge between "I want agents that DO things" and "I don't want to build all the orchestration myself."

---

## Immediate Next Steps

### Quick Wins (Do Now)
- [ ] Fill in `USER.md` with Mark's profile (timezone, preferences, context)
- [ ] Fill in `TOOLS.md` with environment specifics (SSH hosts, Tailscale IPs, device names)
- [ ] Add basic heartbeat tasks to `HEARTBEAT.md`
- [ ] Test the task handoff pattern (write a file in CC, have OpenClaw read it)

### Medium Term
- [ ] Explore Antfarm installation and test a workflow
- [ ] Set up a shared `tasks/` directory convention
- [ ] Configure OpenClaw memory maintenance (MEMORY.md curation during heartbeats)
- [ ] Switch OpenClaw primary model to Haiku for cost optimization

### Longer Term
- [ ] Build custom OpenClaw skills for Mark's specific workflows
- [ ] Explore Claude Agent SDK for programmatic automation
- [ ] Set up Windows PC Ollama firewall rules
- [ ] Consider Docker for sandbox isolation
- [ ] Evaluate multi-agent routing (different agents for different tasks)

---

## Key Insight

**Don't try to replace one system with the other.** Claude Code and OpenClaw serve different purposes:

- **Claude Code** = your power tool (coding, analysis, free inference via Max plan)
- **OpenClaw** = your always-on assistant (messaging, browser, proactive tasks, real-time responses)
- **Shared filesystem** = the bridge between them

The combined system is more capable than either one alone. Use each for what it's best at, share context through files, and let the strengths complement each other.
