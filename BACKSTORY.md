# OpenClaw Project Backstory

Last Updated: 2026-02-02

---

## Session: 2026-02-02 (2) - LAWS Automation via Hook

### What We Did
Fixed the LAWS system failure where Claude wasn't reading BACKSTORY.md at session start despite explicit instructions. Implemented automatic context injection via Claude Code's SessionStart hook system.

### The Problem
LAWS v1 and v2 relied on behavioral compliance - instructions in CLAUDE.md told Claude to read BACKSTORY.md before responding. This failed repeatedly because:
- Instructions are suggestions, not enforcement
- No mechanism to guarantee execution before first output
- User had to start multiple sessions to verify compliance

### The Solution: SessionStart Hook
Implemented automatic BACKSTORY.md injection using Claude Code's hook system:

| Component | Location | Purpose |
|-----------|----------|---------|
| Hook script | `~/.claude/hooks/load-backstory.sh` | Reads BACKSTORY.md from $PWD, outputs to stdout |
| Hook config | `~/.claude/settings.json` (hooks section) | Triggers on startup/resume/compact |

**How it works:**
1. Session starts → Hook fires automatically (no Claude behavior needed)
2. Hook reads `$PWD/BACKSTORY.md` if it exists
3. Hook outputs content to stdout → injected into Claude's context
4. Claude sees `=== BACKSTORY.md (auto-loaded by LAWS hook) ===` in context
5. Claude outputs acknowledgment as first response

### Files Created/Modified
- `~/.claude/hooks/load-backstory.sh` - New hook script (chmod +x)
- `~/.claude/settings.json` - Added hooks.SessionStart configuration
- `~/.claude/CLAUDE.md` - Simplified LAWS section (now just acknowledgment instruction)
- `BACKSTORY.md` - Updated Problems Solved section with v3 solution

### Key Technical Details
Hook configuration in settings.json:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [
          {
            "type": "command",
            "command": "/Users/mark/.claude/hooks/load-backstory.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### Next Steps
- [ ] Start new session to verify hook works
- [ ] Continue with Anthropic auth setup after hook verified

### Cross-References
- Claude Code hooks documentation (researched via claude-code-guide agent)
- Global CLAUDE.md version history updated with LAWS v3

---

## Session: 2026-02-02 - Telegram Setup & LAWS System

### What We Did
Established LAWS system for Claude Code accountability, deep-dived into OpenClaw's value proposition, configured Telegram as the first messaging channel, and set up the development environment with Node 22.

### Key Decisions
- **LAWS system for CLAUDE.md:** Created mandatory acknowledgment system at session start
  - Reason: I failed to read BACKSTORY.md at session start despite instructions existing
  - Solution: LAWS section at top of CLAUDE.md with required visible acknowledgment format
  - Format: `📋 LAWS Executed: ✓ BACKSTORY.md: [status] - [brief context]`

- **Telegram as first channel:** Chosen over WhatsApp/Signal for initial testing
  - Reason: Safest option - Bot API blocks unsolicited messages, no contact list access, no new phone number needed
  - Bot created via @BotFather, completely isolated from personal Telegram

- **Token storage via file:** Using `~/.openclaw/telegram-token.txt` instead of env var or config
  - Reason: Keeps token out of version-controlled config and chat history
  - Config references: `channels.telegram.tokenFile`

- **Gateway auth token:** Generated random token for local gateway auth
  - Reason: OpenClaw now requires auth by default even on loopback
  - Token: stored in `~/.openclaw/openclaw.json` under `gateway.auth.token`

### OpenClaw Value Proposition (vs Claude Code)

| Feature | Claude Code | OpenClaw |
|---------|-------------|----------|
| Availability | Session-based (you open it) | Always-on daemon (reachable anytime) |
| Channels | Terminal only | WhatsApp, Telegram, Signal, Slack, Discord, etc. |
| Agents | One session = one context | Multiple isolated agents with routing |
| Browser | None | CDP-controlled dedicated browser |
| Voice | None | Wake word + push-to-talk |
| Visual output | Text only | Canvas/A2UI for dashboards |
| Mobile | Terminal app | iOS/Android companion apps |
| Parallel work | Linear | Sub-agents for background tasks |

### Multi-Agent Routing (Key Understanding)
- Routing is **configuration-driven, not message-parsed**
- Cannot address agents by name in messages (e.g., `@InsuranceBot`)
- Options for targeting different agents:
  1. Bind specific contacts/groups to agents via config
  2. Different channels for different agents
  3. **Broadcast groups** - multiple agents respond to same message (board of advisors model)
- Agent expertise comes from workspace files: SOUL.md, AGENTS.md, IDENTITY.md

### Problems Solved

1. **BACKSTORY.md not read at session start**
   - Problem: Instruction existed in CLAUDE.md but wasn't followed
   - Solution v1: Created LAWS system with mandatory visible acknowledgment
   - Solution v2: Made instructions more explicit with stop signs
   - **Solution v3 (FINAL):** SessionStart hook auto-injects BACKSTORY.md content
     - Hook: `~/.claude/hooks/load-backstory.sh`
     - Config: `~/.claude/settings.json` (hooks.SessionStart)
     - Fires on: startup, resume, compact
     - Now Claude sees BACKSTORY.md content automatically - no behavioral compliance needed

2. **Node version mismatch (needed 22+, had 20)**
   - Problem: OpenClaw requires Node 22.12.0+, system had 20.17.0
   - Solution: `brew install node@22`, added to PATH in ~/.zshrc
   - Note: PATH export needs `source ~/.zshrc` or new terminal to take effect

3. **pnpm not installed**
   - Solution: `npm install -g pnpm`

4. **Build failed with native binding errors**
   - Problem: Dependencies installed with Node 20, then tried to build with Node 22
   - Solution: `rm -rf node_modules && pnpm install` with Node 22 active

5. **Gateway auth token required**
   - Problem: "Gateway auth is set to token, but no token is configured"
   - Solution: Added `gateway.auth.token` to config with generated random token

6. **Telegram token file in wrong location**
   - Problem: User created file in current directory, not ~/.openclaw/
   - Solution: `cp ./telegram-token.txt ~/.openclaw/telegram-token.txt`

7. **Config format errors**
   - Problem: `gateway.auth: "none"` - expected object, not string
   - Solution: Use object format `gateway.auth: { token: "..." }` or remove entirely

### Files Created/Modified
- `~/.claude/CLAUDE.md` - Added LAWS section with mandatory acknowledgment
- `~/.openclaw/openclaw.json` - Gateway config with Telegram, auth token
- `~/.openclaw/workspace/SOUL.md` - Basic agent persona
- `~/.openclaw/workspace/AGENTS.md` - Agent instructions
- `~/.openclaw/telegram-token.txt` - Telegram bot token (chmod 600)
- `~/.config/ghostty/config` - Added `ctrl+enter=text:\x0a` for newlines

### Current Status
- ✅ Gateway running (LaunchAgent installed, pid active)
- ✅ Telegram channel connected (State: OK)
- ✅ Node 22 installed and in PATH
- ✅ Development environment ready
- ⏳ Anthropic auth not configured (need `pnpm openclaw models auth add --provider anthropic`)
- ❌ First message failed: "No API key found for provider"

### Technical Details
- Config location: `~/.openclaw/openclaw.json`
- Gateway: ws://127.0.0.1:18789 (loopback)
- Telegram user ID: 8242010151 (allowlisted)
- Gateway token: 6d9e8fa2d6049ada3b7ee4eac447f97fe0e4c2c368f2ca511515db3b33d7d374
- LaunchAgent: `~/Library/LaunchAgents/ai.openclaw.gateway.plist`

### Commands Reference
```bash
# Gateway management
pnpm openclaw gateway run      # Start gateway
pnpm openclaw gateway restart  # Restart gateway
pnpm openclaw status           # Check status
pnpm openclaw logs --follow    # Watch logs

# Auth setup (run in terminal, needs TTY)
pnpm openclaw models auth add --provider anthropic
```

### Next Steps
- [ ] Complete Anthropic OAuth setup (`pnpm openclaw models auth add --provider anthropic`)
- [ ] Test Telegram bot responds to messages
- [ ] Explore multi-agent configuration for specialized agents
- [ ] Set up "board of advisors" model with broadcast groups
- [ ] Consider additional channels (WhatsApp, Signal) after Telegram works

### Cross-References
- Global Claude config: `~/.claude/CLAUDE.md` (LAWS system added)
- OpenClaw docs: https://docs.openclaw.ai
- Ghostty config: `~/.config/ghostty/config`

---

## Session: 2026-02-01 - OpenClaw Discovery & Clean Setup

### What We Did
Discovered and evaluated OpenClaw (formerly Clawdbot/Moltbot) as foundation for building an extensive AI assistant system. Identified naming confusion from project renames and initiated clean setup with the latest upstream repo.

### Key Decisions
- **Clean start with latest repo:** Removed outdated `clawdbot` fork pointing to old `moltbot/moltbot` upstream. Will fork fresh from `openclaw/openclaw` to avoid naming confusion and get latest codebase.
- **Project rename history identified:** clawdbot → moltbot → openclaw (current)
- **Goal: Extensive AI assistant solution:** User plans to build comprehensive system, wants clean foundation

### What is OpenClaw?
A **personal AI assistant** you run on your own devices with these key features:
- **Local-first Gateway:** Single control plane for sessions, channels, tools, events
- **Multi-channel inbox:** WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, BlueBubbles, Microsoft Teams, Matrix, Zalo, WebChat
- **Multi-agent routing:** Route inbound channels/accounts/peers to isolated agents
- **Voice Wake + Talk Mode:** Always-on speech for macOS/iOS/Android with ElevenLabs
- **Live Canvas:** Agent-driven visual workspace with A2UI
- **Browser control:** Dedicated managed Chrome/Chromium with CDP control
- **Companion apps:** macOS menu bar app + iOS/Android nodes
- **Skills platform:** Bundled, managed, and workspace skills

### Installation Process (High-Level)
1. **Prerequisites:** Node.js 22+
2. **Install:** `npm install -g clawdbot@latest` (package name may change to openclaw)
3. **Onboard:** `clawdbot onboard --install-daemon` (wizard for gateway, workspace, channels, skills)
4. **Config location:** `~/.clawdbot/clawdbot.json`
5. **Workspace default:** `~/clawd`
6. **Gateway port:** 18789 (default)

### Architecture Overview
```
WhatsApp / Telegram / Slack / Discord / etc.
               │
               ▼
┌───────────────────────────────────┐
│            Gateway                │
│       (control plane)             │
│     ws://127.0.0.1:18789          │
└──────────────┬────────────────────┘
               │
               ├─ Pi agent (RPC)
               ├─ CLI (clawdbot …)
               ├─ WebChat UI
               ├─ macOS app
               └─ iOS / Android nodes
```

### Key Configuration
- **Agent workspace:** `agents.defaults.workspace` (default `~/clawd`)
- **Bootstrap files:** AGENTS.md, SOUL.md, TOOLS.md, IDENTITY.md, USER.md
- **Skills locations:** Bundled, `~/.clawdbot/skills`, `<workspace>/skills`
- **Model recommendation:** Anthropic Claude Opus 4.5 for long-context + prompt-injection resistance

### Problems Encountered
1. **Outdated fork:** User's fork `emmalone/clawdbot` pointed to old `moltbot/moltbot` upstream
   - Solution: Remove old repo, fork fresh from `openclaw/openclaw`

2. **Session working directory deleted:** After removing old clawdbot directory, bash commands failed
   - Solution: User needs to start new Claude Code session after cloning fresh repo

### Files/Repos Referenced
- **Old fork (deleted):** `emmalone/clawdbot` → pointed to `moltbot/moltbot`
- **Current upstream:** `openclaw/openclaw`
- **Related repos discovered:**
  - `openclaw/clawhub` - Skill Directory
  - `VoltAgent/awesome-openclaw-skills` - Skill collection
  - `cloudflare/moltworker` - Run on Cloudflare Workers
  - `NevaMind-AI/memU` - Memory for proactive agents
  - `openclaw/nix-openclaw` - Nix packages

### Documentation Read
- README.md (comprehensive overview)
- AGENTS.md (repository guidelines)
- docs/concepts/architecture.md (Gateway WebSocket architecture)
- docs/cli/setup.md (CLI setup reference)
- docs/cli/onboard.md (onboarding wizard)
- docs/concepts/agent.md (agent runtime, workspace, bootstrap)

### Next Steps
- [ ] Fork `openclaw/openclaw` to `emmalone` account
- [ ] Clone fresh fork to `/Users/mark/PycharmProjects/openclaw`
- [ ] Start new Claude Code session in the openclaw directory
- [ ] Run `pnpm install` to set up development environment
- [ ] Run `clawdbot onboard` to configure personal instance
- [ ] Read remaining documentation for deeper understanding
- [ ] Plan extensive AI assistant solution architecture

### Cross-References
- Global Claude config: `~/.claude/CLAUDE.md`
- OpenClaw docs: https://docs.clawd.bot (may change to openclaw domain)
- Skills registry: https://ClawdHub.com

---

**Note:** This file should be moved to the new `openclaw` project directory after cloning.
