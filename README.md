# Claude Evolving Skills

[中文版](README.zh.md)

Skills that make Claude Code improve itself. Instead of manually chasing viral config repos, these skills scan, evaluate, debate, and adopt improvements automatically.

> **For AI agents**: read [AGENTS.md](AGENTS.md) instead of this file.
>
> Inspired by Karpathy's [Software 3.0](https://www.latent.space/p/s3) talk — this repo is structured for both human and agent consumption. See also the [agents.md spec](https://agents.md/).

## What This Is

A set of reusable skills for Claude Code's `~/.claude/skills/` system. Drop them in and your agent starts improving its own configuration:

- **Scans** GitHub, HN, Reddit, and official vendor docs for new tools and techniques
- **Evaluates** findings with scoring rubrics and multi-model debate (Claude + Gemini + Codex)
- **Reflects** on past sessions to find failure patterns and efficiency improvements
- **Adopts** safe changes automatically, flags risky ones for your review
- **Tracks** every change in a tree-structured history for backtracking

The approach draws from recent self-evolving agent research: [AFlow](https://arxiv.org/abs/2410.10762) (ICLR 2025), [AgentEvolver](https://arxiv.org/abs/2511.10395), [Live-SWE-agent](https://arxiv.org/abs/2511.13646), [EvoAgentX](https://arxiv.org/abs/2507.03616).

## Skills

| Skill | Agents | What it does |
|-------|--------|-------------|
| [agentic-radar](skills/agentic-radar/SKILL.md) | 5 | Scans GitHub trending, HN, Reddit for agentic tools. Updates a local tools registry, proposes CLAUDE.md improvements. |
| [vendor-docs-radar](skills/vendor-docs-radar/SKILL.md) | 3 | Monitors Anthropic, Google, OpenAI official blogs/docs for new features and breaking changes. |
| [reflect-and-learn](skills/reflect-and-learn/SKILL.md) | 6 | The core self-improvement loop. Reviews past sessions, dual-channel scoring, multi-model debate, memory consolidation, experience stripping, tool co-evolution detection, evolution tree tracking. |
| [gemini-agent](skills/gemini-agent/SKILL.md) | — | Delegates a prompt to Gemini CLI. Used by other skills for multi-voice consultation. |
| [codex-agent](skills/codex-agent/SKILL.md) | — | Delegates a prompt to Codex CLI. Used by other skills for multi-voice consultation. |

## Quick Start

### Option A: Let your agent digest this repo (recommended)

The best way to use this repo is to let your AI agent read it and build its own version. Point Claude Code at this repo and tell it:

> Read https://github.com/PalmDr/claude-evolving-skills — understand the skill patterns, then build me a self-evolving setup adapted to my workflow.

Your agent will read the SKILL.md files, understand the patterns (parallel agents, multi-voice debate, evolution tracking), and create skills tailored to what you actually do. This is more aligned with the spirit of the project than copy-pasting someone else's config.

### Option B: Clone and install directly

```bash
# 1. Copy skills into your Claude Code config
git clone https://github.com/PalmDr/claude-evolving-skills.git
cp -r claude-evolving-skills/skills/* ~/.claude/skills/

# 2. (Optional) Copy automation scripts
mkdir -p ~/.claude/scripts
cp claude-evolving-skills/scripts/* ~/.claude/scripts/
chmod +x ~/.claude/scripts/*.sh

# 3. Initialize git in ~/.claude (if not already a repo)
cd ~/.claude && git init

# 4. Initialize evolution tracking
mkdir -p ~/.claude/evolution-history/reflections
touch ~/.claude/evolution-history/evolution-tree.jsonl
touch ~/.claude/evolution-history/scoreboard.jsonl
touch ~/.claude/CHANGELOG.md

# 5. (Optional) Initialize tools registry for agentic-radar
mkdir -p ~/.claude/tools-registry/reviews
touch ~/.claude/tools-registry/REGISTRY.md
touch ~/.claude/tools-registry/adoption-log.md

# 6. (Optional) Initialize report directories for radar skills
mkdir -p ~/.claude/agentic-radar-reports
mkdir -p ~/.claude/vendor-docs-reports

# 7. Add to your CLAUDE.md (or let the skills do it for you)
```

Skills activate when triggered by matching phrases or slash commands.

## Prerequisites

| Tool | Install | Needed for |
|------|---------|------------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `npm i -g @anthropic-ai/claude-code` | All skills |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | `npm i -g @google/gemini-cli` | gemini-agent, reflect-and-learn |
| [Codex CLI](https://github.com/openai/codex) | `npm i -g @openai/codex` | codex-agent, reflect-and-learn |
| git | Pre-installed on most systems | Evolution tracking, committing config changes |
| [jq](https://jqlang.github.io/jq/) | `brew install jq` / `apt install jq` | reflect-and-learn (session JSONL parsing) |

API keys: `GEMINI_API_KEY` (or OAuth) and `OPENAI_API_KEY`.

Gemini and Codex are optional — reflect-and-learn works without them but loses multi-voice debate.

**Cost:** running multi-model debate weekly costs roughly $5-10/month in API calls. Scouting skills use mostly free-tier web search.

## Weekly Automation

All skills can run on a weekly schedule via launchd (macOS) or cron (Linux):

| Day | Skill | Purpose |
|-----|-------|---------|
| Mon | agentic-radar | GitHub + community scan |
| Wed | reflect-and-learn | Session analysis + self-improvement |
| Fri | vendor-docs-radar | Official vendor docs |

### macOS (launchd)

```bash
# Example plist — adjust day/time to your preference
cat > ~/Library/LaunchAgents/com.claude.reflect-and-learn.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude.reflect-and-learn</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-l</string>
        <string>-c</string>
        <string>~/.claude/scripts/reflect-and-learn.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key><integer>3</integer>
        <key>Hour</key><integer>3</integer>
        <key>Minute</key><integer>0</integer>
    </dict>
    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
EOF
launchctl load ~/Library/LaunchAgents/com.claude.reflect-and-learn.plist
```

### Linux (cron)

```cron
0 3 * * 1 ~/.claude/scripts/agentic-radar.sh
0 3 * * 3 ~/.claude/scripts/reflect-and-learn.sh
0 3 * * 5 ~/.claude/scripts/vendor-docs-radar.sh
```

## How It Works

```
┌───────────────┐  ┌─────────────────┐  ┌──────────────────┐
│ agentic-radar │  │ vendor-docs-    │  │ reflect-and-     │
│ (Mon)         │  │ radar (Fri)     │  │ learn (Wed)      │
└───────┬───────┘  └────────┬────────┘  └────────┬─────────┘
        │                   │                    │
        └─────────┬─────────┘                    │
                  ▼                              ▼
        ┌─────────────────┐           ┌──────────────────┐
        │ Proposals       │           │ Session Analysis │
        │ (new tools,     │           │ (failures,       │
        │  config diffs)  │           │  efficiency,     │
        └────────┬────────┘           │  satisfaction)   │
                 │                    └────────┬─────────┘
                 └───────────┬────────────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Multi-Voice Debate   │
                  │ Claude + Gemini +    │
                  │ Codex score & argue  │
                  └──────────┬───────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Adoption             │
                  │ P0/P1 → auto-apply   │
                  │ HIGH-IMPACT → flag   │
                  │ DEFER → log only     │
                  └──────────┬───────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Evolution Tree       │
                  │ (git commit +        │
                  │  tree.jsonl +        │
                  │  scoreboard.jsonl)   │
                  └──────────────────────┘
```

## License

MIT
