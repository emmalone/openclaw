# OpenClaw Project Backstory

Last Updated: 2026-02-11
Sessions: 12 (full log: `6. Agent Output/sessions/openclaw.md` in Obsidian vault)

## Project Identity

OpenClaw is Mark's personal AI gateway — always-on agent system with Telegram, browser control, and multi-channel messaging. Running locally on Mac with Anthropic Sonnet 4.5 as primary model. The agent persona is "Anthony" (Chief of Staff).

## Current State

- Gateway running via LaunchAgent on port 18789 (loopback, token auth)
- Telegram connected: `@anthony_cos_bot`, user allowlist active
- Browser relay extension installed (port 18792), managed browser on port 18800
- Anthropic models working: Sonnet 4.5 (primary), Opus 4.5 (fallback), Haiku 4.5 (available)
- Ollama incompatible with OpenClaw (use directly via CLI only)
- Security hardened: exec approval, mDNS off, session isolation, log redaction, SOUL.md injection defense
- Perplexity Pro search configured (replaced Brave)
- Skills allowlisted to 3: github, obsidian, summarize
- Workspace files condensed (57% token reduction)
- Obsidian vault integration live: `6. Agent Output/` staging, `7. Pending/` task handoff
- Two-tier BACKSTORY system live: lean summary here, full log in vault

## Last Session (2026-02-11, #12)

**What changed:** Implemented two-tier BACKSTORY redesign. Migrated 11 sessions of history to Obsidian vault. Rewrote `/savenow` skill v2.0 with vault log + Anthony decision sync. 96% token reduction per session.
**Unresolved:** Test `/savenow` in a project without existing BACKSTORY.md

## Active Threads

- [ ] Test Anthony via Telegram with condensed workspace files
- [ ] Test Perplexity search via Telegram
- [ ] Phase 2: Test research cycle — Anthony writes output to vault
- [ ] Phase 3: Test task handoff from Claude Code -> OpenClaw
- [ ] Add heartbeat tasks to HEARTBEAT.md
- [ ] Explore Antfarm installation
- [ ] Consider switching primary model to Haiku for cost optimization

## Quick Reference

- Config: `~/.openclaw/openclaw.json` (chmod 600, both tokens literal)
- Workspace: `~/.openclaw/workspace/` (AGENTS.md, SOUL.md, IDENTITY.md, USER.md, TOOLS.md, MEMORY.md)
- Session logs: `~/.openclaw/agents/main/sessions/*.jsonl`
- Gateway logs: `~/.openclaw/logs/gateway.log`
- `openclaw gateway restart` regenerates LaunchAgent plist — use `launchctl bootout/bootstrap` to preserve plist edits
- Forked repo: `emmalone/openclaw`, upstream: `openclaw/openclaw`
