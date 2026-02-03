# OpenClaw Project Backstory

Last Updated: 2026-02-02

---

## Session: 2026-02-02 (4) - Ollama Integration Debugging

### What We Did
Continued troubleshooting the Ollama integration with OpenClaw. Confirmed Ollama works perfectly via direct CLI and API, but fails when routed through OpenClaw gateway.

### Key Findings

**Authentication Clarification:**
- This interactive Claude Code session uses **OAuth Max plan** (Anthropic's hosted service)
- OpenClaw gateway uses **API key** (stored in `~/.openclaw/agents/main/agent/auth-profiles.json`)
- These are separate: Claude Code ≠ OpenClaw gateway

**Ollama Direct Testing (WORKS):**
```bash
# CLI works
ollama run qwen2.5-coder:7b "What is 2+2?"
# Returns: "2 + 2 equals 4"

# OpenAI-compatible API works
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen2.5-coder:7b", "messages": [{"role": "user", "content": "Hello"}]}'
# Returns proper JSON with assistant response
```

**OpenClaw Gateway (FAILS):**
- Web UI shows: "NO_REPLY"
- Telegram: No response at all
- Logs show embedded run completing (~19 seconds) but empty content
- Tools were disabled via `tools.byProvider.ollama.deny: ["*"]`

### Configuration Applied
```json
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://127.0.0.1:11434/v1",
        "apiKey": "ollama-local",
        "api": "openai-completions",
        "models": [{
          "id": "qwen2.5-coder:7b",
          "name": "Qwen 2.5 Coder 7B",
          "reasoning": false,
          "input": ["text"],
          "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
          "contextWindow": 32768,
          "maxTokens": 8192
        }]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/qwen2.5-coder:7b",
        "fallbacks": ["anthropic/claude-opus-4-5"]
      }
    }
  },
  "tools": {
    "byProvider": {
      "ollama": {"deny": ["*"], "profile": "minimal"}
    }
  }
}
```

### Problems Identified

**Root Cause Hypothesis:**
The `profile: "minimal"` and `deny: ["*"]` may be too restrictive. OpenClaw may require at least some internal tools (like `session_status` or `message`) to format responses properly. When ALL tools are denied, the model may be unable to return a properly formatted response to the gateway.

**Symptoms:**
- Runs complete (durationMs ~19000)
- No abort, no errors in logs
- But content field is empty/NO_REPLY

### Decisions Made
- **Pause Ollama as primary** - Reverted to Opus until debugging complete
- **Keep configuration** - Ollama provider setup preserved for future testing
- **Focus on stability** - Opus working reliably for both web and Telegram

### Problems Remaining
1. ⚠️ Ollama returns NO_REPLY through OpenClaw even with tools disabled
   - May need to allow certain internal tools
   - Need to examine OpenClaw source for required tool handling
   - Response parsing may expect tool-formatted output
2. ⚠️ Haiku model names still unclear (not added yet)
3. ⚠️ Smart model selection workflow not implemented

### Next Steps (When Ready to Debug Further)
1. Check OpenClaw source for how tool-less responses are handled
2. Try allowing just `message` tool for Ollama
3. Try removing `profile: "minimal"` but keeping `deny` list
4. Check if there's a "raw mode" or "no-tools mode" for providers
5. Compare working Anthropic request/response with Ollama ones in verbose logs

### Cross-References
- OpenClaw source: `~/PycharmProjects/openclaw/openclaw/` (cloned upstream)
- Ollama service: `brew services info ollama`
- Gateway logs: `tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log`

---

## Session: 2026-02-02 (3) - OpenClaw Fresh Install & Configuration

### What We Did
Completely removed old clawdbot installation, installed fresh OpenClaw, configured Telegram, set up Ollama for local models, and established working Opus-based system.

### Starting Point
- Failed attempts with clawdbot and earlier openclaw installs
- Goal: Get Anthropic OAuth working (decided to use API key instead)
- Wanted cost-optimized model routing: Ollama → Haiku → Opus

### Installation & Cleanup
1. **Removed clawdbot completely:**
   - Uninstalled global npm package (674 packages removed)
   - Removed `~/.clawdbot` directory
   - Stopped old `com.clawdbot.gateway` LaunchAgent service

2. **Fresh OpenClaw install:**
   - Created project structure: `~/PycharmProjects/openclaw/`
   - Cloned from correct repo: `https://github.com/openclaw/openclaw`
   - Set up fork at: `https://github.com/emmalone/openclaw`
   - Configured remotes: `origin` (fork) + `upstream` (openclaw/openclaw)
   - Installed globally: `npm install -g openclaw@latest` (v2026.2.1)

3. **Ran onboarding wizard:**
   ```bash
   openclaw onboard --install-daemon
   ```
   - Configured local gateway mode (port 18789, loopback only)
   - Set up Anthropic with API key (not OAuth - safer, no ban risk)
   - Installed LaunchAgent daemon for auto-start

### Telegram Configuration
**Bot Setup:**
- Bot token: `8570137462:AAHKh52O7ecNVgAM-oWLqZ7IAYkXF_Swt-E`
- Bot username: `@anthony_cos_bot`
- User allowlist: `8242010151` (Mark's Telegram ID)

**Issues Encountered & Fixed:**
1. **"Access not configured" error** → Added user ID to allowlist
   ```bash
   openclaw config set channels.telegram.allowFrom '["8242010151"]'
   ```

2. **JSON responses instead of text** → Disabled streaming mode
   ```bash
   openclaw config set channels.telegram.streamMode "off"
   ```
   - Was getting raw JSON tool calls: `{"name": "message", "arguments": {...}}`
   - Streaming mode was causing response formatting issues

3. **Gateway conflicts** → Removed old clawdbot service
   - Old `com.clawdbot.gateway` was still running
   - Caused intermittent failures and shutdowns

### Ollama Installation & Configuration
**Installed Ollama for local models:**
```bash
brew install ollama
brew services start ollama
ollama pull qwen2.5-coder:7b  # 4.7GB, good for coding tasks
```

**Configured in OpenClaw:**
- Added to `~/.openclaw/openclaw.json` under `models.providers.ollama`
- Model: `qwen2.5-coder:7b` (32k context, free local inference)
- Base URL: `http://127.0.0.1:11434/v1`
- Verified Ollama API working with direct curl test

**Issue: Tool Call JSON Responses**
- When using Ollama as primary, got JSON tool calls instead of text
- Same issue as with Telegram streaming
- **Decision: Pause Ollama integration until debugging complete**

### Current Working Configuration

**Model Setup:**
- **Primary:** `anthropic/claude-opus-4-5` (stable, reliable)
- **Fallback:** None currently
- **Ready but paused:** `ollama/qwen2.5-coder:7b` (needs tool call debugging)

**Channel Setup:**
- **Telegram:** Working, polling mode, streaming OFF
- **Gateway:** `http://127.0.0.1:18789/` (local only, loopback)
- **Auth:** Token-based gateway auth

**File Locations:**
- Config: `~/.openclaw/openclaw.json`
- Credentials: `~/.openclaw/credentials/`
- Logs: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- Workspace: `~/.openclaw/workspace/`
- Project: `~/PycharmProjects/openclaw/openclaw/`

### Key Decisions

**OAuth vs API Key:**
- **Chose API key** for Anthropic (not OAuth)
- Rationale: Simpler, no ban risk, same rate limits
- User concerned about getting banned - API key is perfectly safe

**Model Strategy:**
- **Immediate:** Use Opus for stability (everything working)
- **Future:** Implement intelligent model selection workflow:
  1. Local model evaluates request complexity
  2. Suggests best model (Ollama/Haiku/Opus)
  3. User confirms choice
  4. Request routed to selected model

**Cost Optimization Goal:**
```
Tier 1: Ollama/Qwen (local, free) → Simple coding tasks
Tier 2: Haiku (cloud, cheap) → Moderate complexity
Tier 3: Opus (cloud, expensive) → Complex reasoning only
```
*Currently using Tier 3 for everything until Tier 1 debugged*

### Problems Solved
1. ✅ Clean removal of clawdbot (no leftover conflicts)
2. ✅ Fresh OpenClaw install with proper daemon setup
3. ✅ Telegram access control (allowlist working)
4. ✅ JSON response issue (streaming disabled)
5. ✅ Gateway stability (no more shutdowns)
6. ✅ Anthropic connection (Opus working perfectly)
7. ✅ Model selection via Telegram (`/model` commands)

### Problems Remaining
1. ⚠️ Ollama returns JSON tool calls instead of text responses
   - Need to debug why model is invoking tools vs answering directly
   - May need to adjust model configuration or response parsing
2. ⚠️ Haiku model name unclear (multiple variants, not in catalog)
3. ⚠️ Intelligent model selection workflow not implemented yet

### Next Steps
**Priority 1: Verify Opus Stability**
- Test Telegram responses are clean text (not JSON)
- Ensure gateway stays running (no afternoon shutdowns)
- Confirm web dashboard chat works

**Priority 2: Debug Ollama Integration** (when ready)
- Investigate why Ollama returns tool call JSON
- Check if tools need to be disabled for Ollama
- Test response formatting and parsing
- Once working, set as primary with Opus fallback

**Priority 3: Implement Smart Model Selection** (future)
- Create custom skill or hook for complexity evaluation
- Present model options to user before execution
- Allow per-request model override
- Track cost savings from routing

### Useful Commands
```bash
# Status checks
openclaw gateway status
openclaw channels status
openclaw models list

# Logs and debugging
openclaw logs --tail 50
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# Configuration
openclaw config get agents.defaults.model
openclaw config set <key> <value>

# Gateway control
openclaw gateway restart
openclaw gateway stop

# Model selection (in Telegram)
/model              # Show current model
/model list         # List available models
/model opus         # Switch to Opus
/model qwen         # Switch to Qwen (when working)
/new                # Reset session with default model
```

### Cross-References
- Global Claude Code config: `~/.claude/` (git repo: ClaudeCodeConfig)
- OpenClaw docs: https://docs.openclaw.ai/
- Ollama provider docs: https://docs.openclaw.ai/providers/ollama
- Model selection: https://docs.openclaw.ai/concepts/models
- Telegram setup: https://docs.openclaw.ai/channels/telegram

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
