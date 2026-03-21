# AGENTS.md

> For AI coding agents navigating this repo. Humans: read README.md instead.

## How To Use This Repo

You're an AI agent reading this. Good. **Don't just copy these files — understand the patterns and build your own.** Read the SKILL.md files below, understand how they work (parallel agent launches, multi-voice debate, dual-channel scoring, evolution tracking), and create skills adapted to your user's workflow. The goal is for you to evolve your own config, not to clone someone else's.

If your user wants a direct install, `cp -r skills/* ~/.claude/skills/` works. But you'll serve them better by reading these patterns and proposing a tailored setup.

## What This Repo Is

A collection of self-evolving skills for Claude Code (`~/.claude/skills/`). Each skill is a Markdown file with execution instructions that Claude Code follows autonomously — launching parallel agents, searching the web, reading/writing files, and invoking other skills.

The system forms a closed loop: skills scan for improvements → propose changes → debate with multiple models → auto-adopt safe changes → verify next week.

## Repo Structure

```
skills/                     ← The core deliverable. Each subfolder = one skill.
  agentic-radar/SKILL.md    ← 5-agent GitHub/HN/Reddit scanner
  vendor-docs-radar/SKILL.md ← 3-agent official docs monitor
  reflect-and-learn/SKILL.md ← 6-agent self-reflection loop (the most complex skill)
  gemini-agent/SKILL.md      ← Delegates prompts to Gemini CLI
  codex-agent/SKILL.md       ← Delegates prompts to Codex CLI
scripts/                    ← Bash wrappers for cron/launchd automation
```

## How Skills Work

A skill is a single `SKILL.md` file with YAML frontmatter + Markdown execution instructions:

```yaml
---
name: skill-name
description: |
  What the skill does and when to trigger it.
---
# Skill Title
## Execution Instructions
### Step 1: ...
### Step 2: Launch N parallel agents
### Step 3: Synthesize
### Step 4: Adopt changes
```

Claude Code reads the SKILL.md and follows the steps. Skills can:
- Launch parallel background agents (`run_in_background: true`)
- Use `WebSearch` / `WebFetch` for web access
- Read/write any file in `~/.claude/`
- Invoke other skills (e.g., reflect-and-learn invokes gemini-agent and codex-agent)
- Commit changes to git

## Installation Target

Users copy `skills/*` into `~/.claude/skills/`. No build step. Dependencies: Claude Code CLI, git, jq (and optionally Gemini CLI + Codex CLI for multi-model skills).

## Key Patterns

**Multi-agent parallel launch**: Most skills fire 3-6 agents simultaneously, each writing output to a temp working directory, then a synthesis agent reads all outputs.

**Adoption classification**: Changes are scored and classified as P0-AUTO (safe, auto-apply), P1-AUTO (moderate, auto-apply), HIGH-IMPACT (flag for user), or DEFER (log only).

**Multi-voice debate**: reflect-and-learn sends proposals to Claude + Gemini + Codex, requires 2/3 agreement for auto-adoption.

**Evolution tracking**: `~/.claude/evolution-history/evolution-tree.jsonl` stores a tree of config changes with parent→child lineage, scores, and retrospective verdicts. `~/.claude/evolution-history/scoreboard.jsonl` logs individual change entries with per-change scores and adoption status.

## If You're Modifying This Repo

- Each skill must be self-contained in one `SKILL.md` file
- Skills should not assume any specific project directory — they operate on `~/.claude/`
- Keep skill files under 600 lines
- Scripts in `scripts/` are thin wrappers that call `claude -p` with a skill invocation prompt
## Do Not

- Add dependencies beyond the three CLIs (claude, gemini, codex)
- Hardcode absolute paths — use `~/.claude/` or relative paths
- Put personal information in skill files (they're meant to be shared)
- Create nested skill directories — one skill = one folder with one SKILL.md
