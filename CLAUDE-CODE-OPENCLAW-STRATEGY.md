# Claude Code + OpenClaw: Strategic Architecture Guide

**Author:** Mark Malone (with Claude Code)
**Date:** 2026-02-09
**Goal:** Maximize the Claude Code Max plan, minimize API costs, maintain always-on agent capabilities

---

## The Core Tension

| Resource | Strength | Limitation |
|----------|----------|------------|
| **Claude Code (Max plan)** | Unlimited inference, best coding AI, full system access | Session-based, requires TTY, no daemon mode, no messaging channels |
| **OpenClaw** | Always-on daemon, Telegram/multi-channel, browser CDP, skills | Every interaction requires LLM inference (API key = money) |
| **Ollama** | Free local inference, no API costs | Too weak for OpenClaw's complex tool architecture (tested, documented) |

**The honest truth:** There is no way to route OpenClaw's inference through your Max plan. The Max plan covers Claude Code CLI and claude.ai web only. OpenClaw requires its own API key for every LLM call. These are fundamentally separate auth systems.

---

## What OpenClaw Actually Does (That Requires Inference)

Every Telegram message you send to your OpenClaw bot triggers this chain:

```
You send message → Gateway receives → LLM reads system prompt (4K+ tokens)
→ LLM decides which tools to use → LLM composes response → Response sent back
```

Even a simple "what time is it?" costs ~15K tokens of input (system prompt + tools + context) plus output tokens. There is no "tool-only" mode where OpenClaw executes commands without LLM reasoning.

**What OpenClaw can do WITHOUT inference:** Nothing interactive. The gateway, Telegram polling, browser CDP, and file system are always running — but they're just infrastructure waiting for an LLM to tell them what to do.

---

## What OpenClaw Brings to the Party

### Features You Cannot Replicate with Claude Code Alone

| Feature | OpenClaw | Claude Code | Gap |
|---------|----------|-------------|-----|
| **Always-on messaging** | Telegram, WhatsApp, Signal, Discord, Slack | None — session-based | Large |
| **Browser control (CDP)** | Managed Chrome, screenshot, navigate, interact | None | Large |
| **Voice wake/talk** | ElevenLabs integration, wake word | None | Large |
| **Multi-agent routing** | Multiple isolated agents per channel | Single session | Medium |
| **Session persistence** | Survives restarts, compaction, memory | Lost on exit (BACKSTORY.md workaround) | Medium |
| **Mobile companion apps** | iOS/Android nodes | SSH terminal apps | Small |
| **Canvas/A2UI** | Visual workspace for dashboards | Text output only | Medium |
| **Skills platform** | Installable skills, skill marketplace | Custom slash commands | Small |

### Features Claude Code Already Does Better

| Feature | Claude Code | OpenClaw |
|---------|-------------|----------|
| **Coding** | Best-in-class, full codebase understanding | Basic (designed for assistant tasks) |
| **File operations** | Glob, Grep, Read, Edit, Write — optimized tools | exec-based file ops |
| **Git operations** | Native integration, PR creation, code review | Via exec tool |
| **System automation** | Direct bash, comprehensive tool access | Via exec (with approval) |
| **Cost** | $0 on Max plan | Per-token API costs |
| **Context window** | Automatic compaction, huge context | Limited by model/cost |
| **Plan mode** | Built-in architectural planning | Not available |

---

## Recommended Architecture

### Tier 1: Claude Code via SSH (Primary — Free)

**This is your workhorse.** SSH from anywhere, do everything:

```
iPhone/iPad/Laptop → SSH → Mac Mini → Claude Code
```

**Use for:**
- All coding and development
- File system operations
- Git workflows
- System administration
- Complex research and analysis
- Planning and architecture
- Anything that can wait for you to open a terminal

**Cost:** $0 (Max plan)

**Setup:**
```bash
# From anywhere
ssh mark@your-tailscale-ip
cd ~/PycharmProjects/openclaw
claude
```

### Tier 2: OpenClaw with Haiku (Messaging Bridge — Cheap)

**Use OpenClaw for what Claude Code can't do:** real-time messaging, browser control, and quick mobile interactions.

**Use for:**
- Quick questions from Telegram when you can't SSH
- "Read this URL and summarize it" (browser + LLM)
- Forwarding tweets/articles for later review
- Simple status checks
- Anything that needs to happen while you're away from a terminal

**Cost model with Haiku:**
| Interaction | Input Tokens | Output Tokens | Cost |
|-------------|-------------|---------------|------|
| Simple question | ~15K | ~200 | ~$0.004 |
| URL summary | ~20K | ~500 | ~$0.006 |
| 50 messages/day | ~1M | ~25K | ~$0.28/day |
| Monthly (moderate use) | ~30M | ~750K | ~$8.50/month |

That's roughly **the cost of a coffee per month** for always-on Telegram access.

**Current config change to make Haiku primary for OpenClaw:**
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-haiku-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5"]
      }
    }
  }
}
```

### Tier 3: Ollama Direct (Local Free Inference — Manual)

**Use Ollama directly from the command line**, not through OpenClaw:

```bash
# Quick coding questions
ollama run qwen2.5-coder:14b "explain this error message: ..."

# Code generation
ollama run qwen2.5-coder:14b "write a python script that..."
```

**Why not through OpenClaw:** Documented in sessions 4-5 — local models can't handle OpenClaw's complex system prompt and tool architecture. They either output `NO_REPLY` or raw JSON tool calls.

**Use for:**
- Quick throwaway questions that don't need tool access
- Code snippets and explanations
- Offline work when you have no internet

---

## Can You Replace OpenClaw with Claude Code?

### What You Could Build Yourself

**Simple Telegram relay bot (no LLM needed):**
```python
# Concept: Telegram bot that queues messages for Claude Code
# Bot receives message → writes to ~/queue/inbox/
# Claude Code reads inbox, processes, writes to ~/queue/outbox/
# Bot sends outbox contents back to Telegram
```

This would give you:
- Telegram messaging (free, no API key)
- Claude Code does the thinking (Max plan)
- File-based message queue

**Limitations:**
- Not real-time (needs active Claude Code session to process)
- No browser control
- No streaming responses
- You'd need to build the queue system, error handling, retry logic
- Claude Code can't be triggered by incoming messages automatically

### What You Cannot Replicate

1. **Real-time responses:** Claude Code requires an active session. You can't have it sitting idle waiting for Telegram messages. It would need to poll a queue file, which means:
   - Either you have a Claude Code session always open (wastes Max plan quota, may trigger cooldowns)
   - Or there's a delay until you open a session and process the queue

2. **Browser CDP control:** OpenClaw's managed Chrome with DevTools Protocol is deeply integrated. Building this from scratch is a significant engineering project.

3. **Session persistence across restarts:** OpenClaw maintains conversation context through compaction and session logs. Claude Code sessions are ephemeral.

4. **Multi-channel routing:** Supporting Telegram + WhatsApp + Discord simultaneously with agent routing is what OpenClaw was built for.

### The Honest Assessment

| Approach | Effort | Capability | Cost |
|----------|--------|------------|------|
| **OpenClaw + Haiku** | Already set up | Full messaging + browser + tools | ~$8/month |
| **Custom Telegram relay + Claude Code** | 2-4 days to build | Messaging only, not real-time | $0 |
| **Custom full replacement** | Weeks/months | Could match OpenClaw eventually | $0 inference, huge time cost |
| **Claude Code only (SSH)** | Zero | No messaging, no always-on | $0 |

**Recommendation:** Use OpenClaw with Haiku for the ~$8/month it costs. The engineering time to replicate even half of what OpenClaw provides would cost far more in your time than years of Haiku API costs.

---

## Optimizing OpenClaw for Minimum API Usage

### 1. Use Haiku as Primary Model

Haiku is 60x cheaper than Opus and handles most assistant tasks well:

```bash
openclaw config set agents.defaults.model.primary "anthropic/claude-haiku-4-5"
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-sonnet-4-5"]'
```

### 2. Use /model Command for Escalation

When a task needs more capability, escalate manually in Telegram:
```
/model sonnet    # For moderate complexity
/model opus      # For complex reasoning (use sparingly)
/model haiku     # Switch back when done
```

### 3. Reduce System Prompt Size

The default system prompt is ~4K tokens sent with every request. Consider trimming:
- `~/.openclaw/workspace/AGENTS.md` — keep only essential instructions
- `~/.openclaw/workspace/SOUL.md` — keep concise
- Remove unused bootstrap files if not needed

### 4. Use Short Sessions

Each `/new` command resets context, reducing input tokens on subsequent messages. For unrelated questions, start fresh sessions.

### 5. Offload Heavy Work to Claude Code

When a Telegram conversation reveals complex work:
1. Ask OpenClaw to write a task file: "Write a description of this task to ~/tasks/current.md"
2. SSH into your Mac Mini
3. Open Claude Code, read the task file, execute with full power
4. Tell OpenClaw the result if needed

### 6. Monitor Costs

```bash
# Check recent API usage
openclaw api-size  # if available
# Or check Anthropic dashboard
# https://console.anthropic.com/settings/usage
```

---

## The Integrated Workflow

### Daily Pattern

```
Morning (iPhone, on the go):
  └─ Telegram → OpenClaw (Haiku) → Quick questions, read articles, check status

Work session (SSH from anywhere):
  └─ SSH → Claude Code (Max plan) → Coding, architecture, complex tasks
  └─ If needed: Claude Code writes files → OpenClaw can reference them

Evening (iPhone, casual):
  └─ Telegram → OpenClaw (Haiku) → "Summarize what we did today"
  └─ Forward interesting articles → OpenClaw summarizes → saves markdown

Always running (background):
  └─ OpenClaw gateway → listening on Telegram
  └─ Ollama → available for quick CLI queries
```

### When to Use What

| Scenario | Tool | Why |
|----------|------|-----|
| Writing code | Claude Code (SSH) | Best coding AI, free on Max plan |
| Quick question from phone | OpenClaw (Telegram/Haiku) | Mobile, real-time, cheap |
| "Read this URL" | OpenClaw (browser + Haiku) | Has browser CDP, can fetch pages |
| Complex architecture | Claude Code (SSH) | Plan mode, full context, free |
| "Remind me about X" | OpenClaw (Telegram/Haiku) | Always-on, session memory |
| Git operations | Claude Code (SSH) | Native integration, free |
| Forward a tweet | OpenClaw (browser + Haiku/Sonnet) | Browser logged into X, can read tweets |
| System administration | Claude Code (SSH) | Full system access, free |
| Quick code question (offline) | Ollama (direct CLI) | Free, local, no internet needed |

---

## Implementation Checklist

### Already Done
- [x] Claude Code working via SSH (Max plan)
- [x] OpenClaw gateway running (LaunchAgent)
- [x] Telegram bot configured and working
- [x] Ollama installed with local models
- [x] Security hardening applied
- [x] Brave Search API configured
- [x] Browser control working (CDP, logged into X)

### To Do Now
- [ ] Switch OpenClaw primary model to Haiku (cheapest)
- [ ] Trim workspace bootstrap files for smaller system prompt
- [ ] Test Haiku handles your typical Telegram interactions adequately

### To Do Later
- [ ] Set up Anthropic usage monitoring/alerts
- [ ] Consider custom Telegram relay bot if you want $0 messaging (with delayed responses)
- [ ] Explore Claude Agent SDK for programmatic automation
- [ ] Windows firewall rules for Ollama (security hardening pending)
- [ ] Docker for sandbox isolation (if needed)

---

## Cost Comparison Summary

| Setup | Monthly Cost | Capabilities |
|-------|-------------|--------------|
| Claude Code only (SSH) | $0 (Max plan) | Everything except messaging/always-on |
| + OpenClaw with Haiku | ~$8-15/month | + Telegram, browser, always-on |
| + OpenClaw with Sonnet | ~$50-100/month | + Better reasoning for complex tasks |
| + OpenClaw with Opus | ~$200-500/month | Full capability (unnecessary for most tasks) |
| Custom relay bot | $0 | Telegram with delays, no browser, fragile |

**The sweet spot:** Claude Code (SSH) for heavy work + OpenClaw (Haiku) for messaging = ~$8/month total beyond your Max plan.

---

## Key Takeaway

**OpenClaw and Claude Code serve different purposes and complement each other:**

- **Claude Code** = your power tool (coding, complex tasks, free inference)
- **OpenClaw** = your always-on assistant (messaging, browser, quick tasks)
- **Ollama** = your offline backup (direct CLI only, not through OpenClaw)

Don't try to replace one with the other. Use each for what it's best at. The combined cost is essentially just your Max plan subscription + ~$8/month in Haiku API calls.
