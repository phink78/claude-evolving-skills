---
name: vendor-docs-radar
description: >
  Scans official engineering blogs and documentation from Anthropic (Claude),
  Google (Gemini CLI), and OpenAI (Codex) for new posts, features, best practices,
  and breaking changes — then proposes concrete adoptions to the user's setup.
trigger_phrases:
  - "scan vendor docs"
  - "check official docs"
  - "what's new from Anthropic"
  - "what's new from OpenAI"
  - "what's new from Google Gemini"
  - "run vendor docs radar"
  - "check for new Claude features"
  - "scan official blogs"
  - "vendor docs update"
  - "official docs radar"
---

# Vendor Docs Radar

Scans official engineering blogs and documentation from the three major AI coding
assistant vendors — Anthropic (Claude Code), Google (Gemini CLI), and OpenAI (Codex) —
for new posts, feature announcements, best practices, and breaking changes.
Synthesizes findings and proposes concrete adoptions to the user's Claude Code setup.

## When to Use

- Weekly scheduled runs (pair with agentic-radar for full coverage)
- When the user asks "what's new from Anthropic/Google/OpenAI"
- Before major config changes, to check for official guidance
- When a new version of Claude Code, Gemini CLI, or Codex is released

## Output

- Dated report at `~/.claude/vendor-docs-reports/YYYY-MM-DD.md`
- Proposed adoption items (config changes, new features to enable, patterns to adopt)
- Updated `~/.claude/vendor-docs-reports/CHANGELOG.md` tracking what was found and when

---

## Execution Instructions

### Step 1: Setup

```
mkdir -p ~/.claude/vendor-docs-reports
```

Set `DATE` to today's date (YYYY-MM-DD).

### Step 2: Launch 3 Parallel Scan Agents

Launch ALL three agents simultaneously as background agents with `mode: bypassPermissions`.
Each agent uses WebSearch and WebFetch to find and extract content.

---

#### Agent 1: Anthropic / Claude Official Scanner

**Goal:** Find new blog posts, engineering articles, documentation updates, and
changelog entries from Anthropic related to Claude Code, Claude API, and Claude models.

**Sources to scan (use WebSearch + WebFetch):**

| Source | URL / Search Query |
|--------|-------------------|
| Anthropic Blog | `site:anthropic.com/research OR site:anthropic.com/news` — search for posts from last 30 days |
| Anthropic Engineering | `site:anthropic.com/engineering` |
| Claude Code Docs | `site:docs.anthropic.com/en/docs/claude-code` |
| Claude API Docs | `site:docs.anthropic.com/en/docs/` — check for new/updated pages |
| Anthropic Cookbook | `site:github.com/anthropics/anthropic-cookbook` — recent commits/new notebooks |
| Claude Code GitHub | Fetch `https://github.com/anthropics/claude-code/releases` for new releases |
| Claude Code Changelog | Search `"claude code" changelog OR "what's new" site:anthropic.com` |

**Search queries (run ALL):**
1. `anthropic blog` (current year)
2. `anthropic engineering blog new`
3. `claude code new features` (current year)
4. `docs.anthropic.com claude code updates`
5. `anthropic claude best practices` (current year)
6. `claude code hooks MCP new`
7. `anthropic claude model context protocol updates`
8. `claude code CLI release notes`
9. `anthropic prompt engineering guide updates`
10. `anthropic cookbook new notebooks` (current year)

**Extract for each finding:**
- Title + URL
- Date published/updated
- Category: `blog` | `docs` | `changelog` | `cookbook` | `release`
- One-paragraph summary
- Relevance to our Claude Code setup (HIGH / MEDIUM / LOW)
- Specific actionable items (new flags, config options, best practices)

**Return format:** Markdown table of findings sorted by date (newest first),
followed by a "Key Takeaways" section with actionable items.

---

#### Agent 2: Google Gemini CLI Scanner

**Goal:** Find new documentation, blog posts, release notes, and best practices
for Gemini CLI and Gemini API that could improve Gemini agent integration.

**Sources to scan (use WebSearch + WebFetch):**

| Source | URL / Search Query |
|--------|-------------------|
| Gemini CLI GitHub | Fetch `https://github.com/google-gemini/gemini-cli/releases` for new releases |
| Gemini CLI Docs | `site:github.com/google-gemini/gemini-cli` — README, docs/, wiki |
| Google AI Blog | `site:blog.google/technology/ai` — Gemini related posts |
| Google Developers Blog | `site:developers.googleblog.com gemini` |
| Gemini API Docs | `site:ai.google.dev/gemini-api/docs` |
| Google Cloud AI | `site:cloud.google.com/vertex-ai/generative-ai/docs` — Gemini updates |

**Search queries (run ALL):**
1. `gemini cli new features` (current year)
2. `google gemini cli release notes`
3. `gemini cli best practices`
4. `gemini api updates` (current year)
5. `google gemini developer tools new`
6. `gemini cli configuration options`
7. `gemini pro new capabilities` (current year)
8. `google ai studio gemini updates`
9. `gemini cli extensions plugins`
10. `gemini code assistance tools` (current year)

**Extract for each finding:**
- Title + URL
- Date published/updated
- Category: `blog` | `docs` | `changelog` | `release` | `api-update`
- One-paragraph summary
- Relevance to gemini-agent skill (HIGH / MEDIUM / LOW)
- Specific actionable items (new CLI flags, model updates, API changes)

**Return format:** Same as Agent 1.

---

#### Agent 3: OpenAI Codex CLI Scanner

**Goal:** Find new documentation, blog posts, release notes, and best practices
for OpenAI Codex CLI and OpenAI API that could improve Codex agent integration.

**Sources to scan (use WebSearch + WebFetch):**

| Source | URL / Search Query |
|--------|-------------------|
| Codex CLI GitHub | Fetch `https://github.com/openai/codex/releases` for new releases |
| Codex CLI Docs | `site:github.com/openai/codex` — README, docs/ |
| OpenAI Blog | `site:openai.com/blog` OR `site:openai.com/index` — Codex related |
| OpenAI API Docs | `site:platform.openai.com/docs` — new endpoints, models |
| OpenAI Cookbook | `site:github.com/openai/openai-cookbook` — recent additions |
| OpenAI Changelog | `site:platform.openai.com/docs/changelog` |

**Search queries (run ALL):**
1. `openai codex cli new features` (current year)
2. `openai codex cli release notes`
3. `openai codex best practices`
4. `openai api updates` (current year)
5. `openai developer tools new`
6. `openai codex configuration options`
7. `openai model new capabilities` (current year)
8. `openai platform changelog` (current year)
9. `openai codex cli extensions plugins`
10. `openai coding assistant tools` (current year)

**Extract for each finding:**
- Title + URL
- Date published/updated
- Category: `blog` | `docs` | `changelog` | `release` | `api-update`
- One-paragraph summary
- Relevance to codex-agent skill (HIGH / MEDIUM / LOW)
- Specific actionable items (new CLI flags, model updates, API changes)

**Return format:** Same as Agent 1.

---

### Step 3: Synthesis (after all 3 agents complete)

Launch a synthesis agent that receives all three agent outputs and produces:

#### 3a. Dated Report

Write to `~/.claude/vendor-docs-reports/YYYY-MM-DD.md`:

```markdown
---
date: YYYY-MM-DD
sources_scanned: [anthropic, google-gemini, openai]
---

# Vendor Docs Radar — YYYY-MM-DD

## Executive Summary
<!-- 3-5 bullet points of the most important findings across all vendors -->

## Anthropic / Claude
### New Posts & Docs
<!-- Table of findings from Agent 1 -->
### Actionable Items
<!-- Bulleted list of things to adopt/change -->

## Google / Gemini CLI
### New Posts & Docs
<!-- Table of findings from Agent 2 -->
### Actionable Items

## OpenAI / Codex
### New Posts & Docs
<!-- Table of findings from Agent 3 -->
### Actionable Items

## Cross-Vendor Trends
<!-- Patterns appearing across 2+ vendors — convergent features, shared best practices -->

## Proposed Adoptions
<!-- Specific changes to propose to the user's Claude Code setup -->
| Priority | Source | Change | Rationale |
|----------|--------|--------|-----------|
| HIGH | ... | ... | ... |
| MEDIUM | ... | ... | ... |

## Sources
<!-- All URLs referenced in this report -->
```

#### 3b. Update Changelog

Append to `~/.claude/vendor-docs-reports/CHANGELOG.md`:
```
## YYYY-MM-DD
- Scanned: Anthropic, Google Gemini, OpenAI
- Found: N new items (X high-relevance)
- Key: <one-line summary of most important finding>
```

### Step 4: Execute and Propose Adoptions

Classify each adoption item by safety:

**Auto-execute (P0/P1, safe):** Apply immediately without asking.
- Model name/version updates in skill configs
- New config additions (flags, settings) that are additive
- Language softening or clarification in docs
- New references, URLs, or source links
- Adding new search queries or scan sources to this skill

**Flag for user review (HIGH-IMPACT):** Present as proposals, do NOT auto-apply.
- Changes to execution style (Section 1 of CLAUDE.md)
- Permission or security-related changes
- Removing or replacing existing functionality
- Breaking changes to APIs or CLIs we depend on

For each auto-executed item, log what was changed and why (with source URL).
For each flagged item, present: specific file + diff, source URL, risk level.

### Step 5: Commit All Changes

Commit the report AND any auto-executed adoption changes together:

```bash
cd ~/.claude && git add -A && git commit -m "vendor-docs-radar: scan YYYY-MM-DD + auto-adopt P0/P1" && git push
```

---

## Automation

To run weekly, create a launchd plist at `~/Library/LaunchAgents/com.claude.vendor-docs-radar.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude.vendor-docs-radar</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-l</string>
        <string>-c</string>
        <string>~/.claude/scripts/vendor-docs-radar.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key>
        <integer>5</integer>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/vendor-docs-radar.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/vendor-docs-radar.err</string>
</dict>
</plist>
```

**Script:** `~/.claude/scripts/vendor-docs-radar.sh`
```bash
#!/bin/bash
cd ~/.claude
claude -p "Run the vendor-docs-radar skill. Today is $(date +%Y-%m-%d)."
```

For Linux, use cron instead:
```cron
0 3 * * 5 ~/.claude/scripts/vendor-docs-radar.sh
```

---

## Differences from agentic-radar

| Aspect | vendor-docs-radar | agentic-radar |
|--------|-------------------|---------------|
| **Focus** | Official vendor docs & blogs | Community repos & discussions |
| **Sources** | anthropic.com, ai.google.dev, openai.com, platform.openai.com | GitHub trending, HN, Reddit |
| **Signal type** | Authoritative best practices, new features, breaking changes | Emerging tools, community patterns |
| **Agents** | 3 (one per vendor) + synthesis | 5 (GitHub, community, registry, deep-dive, audit) |
| **Output** | Dated report + adoption proposals | Report + registry updates + CLAUDE.md diffs |
| **Cadence** | Weekly (Fridays) | Weekly (Mondays) |

**Use both together** for full coverage: vendor-docs-radar catches official announcements,
agentic-radar catches community innovations.

---

## Notes

- Prioritize Anthropic findings — that's the primary platform
- Cross-vendor trends (features appearing in 2+ platforms) are especially valuable signals
- When a vendor announces a breaking change, flag it as HIGH priority immediately
- Compare findings week-over-week to track velocity of changes
- If a vendor doc recommends a practice you're not following, that's always HIGH priority
