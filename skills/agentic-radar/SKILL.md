---
name: agentic-radar
description: |
  Self-evolving scanner that discovers trending agentic systems, Claude Code configurations, and AI agent patterns, then proposes concrete improvements to the user's own Claude Code setup. Runs 5 parallel scan agents (GitHub trending, community discussions, registry updates, deep repo analysis, setup audit), synthesizes findings, writes a dated report, updates the tools registry, and proposes specific CLAUDE.md improvements as diffs. Use this for "scan for agentic trends", "run agentic radar", "what's new in Claude Code / AI agents", "audit my setup", or weekly automated runs via launchd.
---

# Agentic Radar

A self-evolving scanner that discovers trending agentic systems, Claude Code configurations, and AI agent patterns across GitHub, HN, and Reddit — then proposes concrete improvements to the user's own Claude Code setup.

## When to Use

- Weekly automated run (via launchd/cron)
- Manually via `/agentic-radar` or "run the agentic radar"
- When the user asks "what's new in Claude Code / AI agents / agentic coding"
- When the user asks "audit my Claude setup" or "how can I improve my Claude config"
- After hearing about a new agentic pattern and wanting to see if there are more like it

## Output

The skill produces:
1. A dated report at `<REPORTS_DIR>/agentic-radar-{YYYY-MM-DD}.md`
2. Updated entries in `~/.claude/tools-registry/REGISTRY.md`
3. Stub reviews for notable repos in `~/.claude/tools-registry/reviews/`
4. Proposed diffs for `~/.claude/CLAUDE.md` (P0/P1 auto-applied; HIGH-IMPACT flagged for user review)
5. Auto-adopted patterns from standout repos (committed to git)

> **Configuration:** Set `REPORTS_DIR` to your preferred report output directory (e.g., `~/reports/agentic-radar`).

---

## Execution Instructions

### Step 0: Setup

```bash
REPORTS_DIR=${REPORTS_DIR:-~/.claude/agentic-radar-reports}
mkdir -p "$REPORTS_DIR"
```

Set `DATE` to today's date (YYYY-MM-DD).
Set `REPORT_PATH` to `$REPORTS_DIR/agentic-radar-{DATE}.md`.
Create a temporary working directory for agent outputs: `$REPORTS_DIR/.agentic-radar-work-{DATE}/`.

### Step 1: Launch 5 parallel scan agents (ALL at once, `run_in_background: true`, `mode: bypassPermissions`)

Fire all 5 simultaneously. Do not wait for one to finish before launching the next.

---

**Agent 1 — GitHub Trending Scanner**

Prompt template:
```
You are scanning GitHub for trending repos related to agentic systems, Claude Code, and AI coding agents from the last 30 days.

Run at least 15 web searches using these queries:
1. GitHub trending repositories "claude code" OR "claude-code" (last month)
2. GitHub repos "agentic" OR "ai agent" coding (sort: stars, pushed: last 30 days)
3. GitHub "CLAUDE.md" template OR config (sort: updated)
4. GitHub "mcp server" claude OR anthropic (sort: stars, pushed: last 30 days)
5. GitHub "claude hooks" OR "claude-code hooks" (sort: updated)
6. GitHub "coding agent" framework 2025 OR 2026 (sort: stars)
7. GitHub "agentic workflow" LLM (sort: stars, pushed: last 3 months)
8. GitHub topic:claude-code OR topic:claude-agents OR topic:mcp-server
9. GitHub "ai coding assistant" framework (sort: stars, pushed: last 30 days)
10. GitHub "agent orchestration" LLM (sort: stars)
11. GitHub awesome-claude OR awesome-mcp OR awesome-agents (sort: updated)
12. GitHub "system prompt" claude agent (sort: stars)
13. GitHub "claude subagent" OR "background agent" claude
14. GitHub "agentic coding" patterns OR best-practices
15. GitHub "claude code" skills OR slash-commands OR extensions

For each repo found, record:
- Full repo name (owner/repo)
- URL
- Stars count
- Last push date
- One-line description
- Category: [claude-config | mcp-server | agent-framework | hooks | skills | workflow | prompts | toolkit | other]
- Relevance score (1-10): How useful is this for improving a Claude Code setup?

Produce a ranked list of the top 30 repos, sorted by relevance score.

Save to: {WORK_DIR}/github_trending.md
```

---

**Agent 2 — Community Discussion Scanner**

Prompt template:
```
You are scanning online communities for discussions about agentic coding, Claude Code tips, and AI agent patterns from the last 30 days.

Run at least 15 web searches:
1. Hacker News "claude code" tips OR tricks OR workflow (last 30 days)
2. Reddit r/ClaudeAI "claude code" setup OR config OR CLAUDE.md
3. Reddit r/ClaudeAI hooks OR skills OR MCP (last 30 days)
4. Hacker News "agentic coding" OR "coding agent" (last 30 days)
5. Reddit r/LocalLLaMA "claude code" OR "coding agent" patterns
6. Hacker News "MCP server" OR "model context protocol" (last 30 days)
7. Twitter/X "claude code" tip OR trick OR workflow (last 30 days)
8. Reddit r/ClaudeAI "background agent" OR "subagent" OR "parallel"
9. Hacker News "AI coding" workflow OR productivity (last 30 days)
10. Reddit r/MachineLearning "agentic" patterns OR framework (last 30 days)
11. "claude code" best practices blog OR article (current year)
12. "agentic system" design patterns blog OR article
13. Hacker News "claude hooks" OR "claude skills"
14. Reddit r/ClaudeAI "settings.json" OR "permissions" OR "hooks"
15. "coding agent" comparison OR benchmark (current year)

For each notable discussion/post, record:
- Title and URL
- Source (HN / Reddit / Blog / Twitter)
- Date
- Key insight or tip (2-3 sentences)
- Actionability: [high | medium | low] — could this directly improve a Claude Code setup?
- Category: [workflow-tip | tool-discovery | pattern | config-trick | anti-pattern | benchmark]

Produce a ranked list of the top 20 most actionable insights.

Save to: {WORK_DIR}/community_discussions.md
```

---

**Agent 3 — Registry Update Scanner**

Prompt template:
```
You are checking for updates to repos already tracked in the user's tools registry.

First, read the current registry at ~/.claude/tools-registry/REGISTRY.md

For each tracked repo (up to 20):
1. Check the repo's current star count, last commit date, and recent releases
2. Note any major version bumps, new features, or breaking changes in the last 30 days
3. Check if the repo has been archived or deprecated

Also search for:
- New repos by the same authors/orgs as tracked repos
- Forks of tracked repos that have diverged significantly
- Repos that reference or extend tracked repos

Produce a report with:
- UPDATED: repos with significant changes (new features, major releases)
- STALE: repos with no commits in 90+ days
- DEPRECATED: repos that are archived or abandoned
- RELATED: new repos from tracked authors or extending tracked repos

Save to: {WORK_DIR}/registry_updates.md
```

---

**Agent 4 — Deep Repo Analyzer**

Prompt template:
```
You are doing deep analysis of the most interesting new repos discovered in agentic AI / Claude Code space.

Search for and identify 3-5 of the most promising new repos (high stars, recent activity, high relevance to Claude Code workflows). For each:

1. Fetch and read the full README
2. Examine the repo structure (key directories and files)
3. Read 2-3 key source files that reveal the core mechanism
4. Check open issues and recent PRs for community engagement signals

For each repo, produce:
- **Name**: owner/repo
- **URL**: full GitHub URL
- **What it does**: 2-3 sentence summary
- **Core mechanism**: How it works technically (architecture, key abstractions)
- **Integration potential**: How it could integrate with a Claude Code ~/.claude/ setup
- **Adoption effort**: [drop-in | moderate | significant] — how hard to adopt
- **Quality signals**: Stars, contributors, test coverage, documentation quality, release cadence
- **Concrete adoption steps**: Numbered list of what to do to start using it
- **Risk factors**: Dependencies, stability, maintenance likelihood

Rank by: Integration potential x Quality signals.

Save to: {WORK_DIR}/deep_repo_analysis.md
```

---

**Agent 5 — Setup Auditor**

Prompt template:
```
You are auditing the user's current Claude Code setup for gaps and improvement opportunities.

Read and analyze these files:
- ~/.claude/CLAUDE.md (global config)
- ~/.claude/settings.json (permissions and settings)
- List all files in ~/.claude/skills/ (installed skills)
- List all files in ~/.claude/hooks/ (if exists)
- List all files in ~/.claude/scripts/ (automation scripts)
- List all files in ~/.claude/methodology/ (research methodology)
- ~/.claude/tools-registry/REGISTRY.md (tracked tools)
- ~/.claude/tools-registry/adoption-log.md (what's been adopted)

Produce an audit report covering:

1. **CLAUDE.md Analysis**
   - What's well-configured
   - Missing sections compared to best practices (e.g., error handling, output formatting, project-specific overrides)
   - Rules that may be outdated or conflicting

2. **Skills Gap Analysis**
   - What skills are installed
   - What common use cases have no skill coverage
   - Skills that could be combined or simplified

3. **Hooks Assessment**
   - What hooks exist (if any)
   - Common hook patterns that are missing (pre-commit validation, auto-formatting, notification hooks)
   - Hooks from the community that could be adopted

4. **MCP Server Inventory**
   - What MCP servers are configured (check settings.json)
   - High-value MCP servers that are missing
   - MCP servers that may be redundant

5. **Automation Gaps**
   - What's automated (cron/launchd jobs)
   - What should be automated but isn't
   - Scripts that could be turned into skills

6. **Security Review**
   - Permission scope in settings.json (too broad? too narrow?)
   - Any sensitive data in committed files
   - Bypass permissions that should be tightened

7. **Concrete Recommendations** (ranked by impact)
   - Top 5 improvements with specific implementation steps
   - Estimated effort for each: [5 min | 30 min | 2 hours]

Save to: {WORK_DIR}/setup_audit.md
```

---

### Step 2: Wait for all 5 agents to complete

As each agent completes, brief the user with a 1-line summary:
- "Agent 1 (GitHub) complete — found N trending repos, top: owner/repo"
- "Agent 2 (Community) complete — N actionable insights, top pattern: X"
- "Agent 3 (Registry) complete — N repos updated, N stale, N new related"
- "Agent 4 (Deep Analysis) complete — analyzed N repos, top pick: owner/repo"
- "Agent 5 (Audit) complete — N improvement opportunities, top gap: X"

Do NOT launch synthesis until all 5 are complete.

### Step 3: Launch synthesis agent (`run_in_background: true`, `mode: bypassPermissions`)

Once all 5 agents are done:

```
Read all files in {WORK_DIR}:
- github_trending.md
- community_discussions.md
- registry_updates.md
- deep_repo_analysis.md
- setup_audit.md

Cross-reference findings across all 5 agents and write a comprehensive report.

## Report Structure

Write to: {REPORT_PATH}

# Agentic Radar Report — {DATE}

## Executive Summary
5 bullets: the most important findings and recommended actions.

## 1. Trending Repos & Tools
Top 10 repos discovered, with:
- Name, URL, stars, category
- Why it matters (1 sentence)
- Integration recommendation: [adopt now | watch | skip]
Organized by category: claude-config, mcp-server, agent-framework, hooks, skills, workflow, toolkit

## 2. Community Pulse
Top 10 community insights/patterns:
- The insight
- Source and URL
- How to apply it to the user's setup

## 3. Registry Health
- Repos with significant updates (act on these)
- Stale repos (consider removing)
- Deprecated repos (remove from registry)
- New related repos to track

## 4. Deep Dives
For each deeply analyzed repo (3-5):
- Full analysis summary
- Concrete adoption steps
- Risk assessment

## 5. Setup Audit Results
- Current setup strengths
- Top 5 improvement opportunities (ranked by impact)
- Specific implementation steps for each

## 6. Proposed Changes

### CLAUDE.md Improvements
Show each proposed change as a diff:
```diff
- old line
+ new line
```
Explain why each change is recommended.

### New Skills to Create
For each proposed skill:
- Name and purpose
- Trigger phrases
- Estimated creation effort

### New Hooks to Add
For each proposed hook:
- Hook type (pre-commit, post-response, etc.)
- What it does
- Implementation sketch

### MCP Servers to Add
For each proposed MCP server:
- Name and URL
- What it provides
- Configuration snippet

### Tools to Adopt
For each tool recommended for adoption:
- Name, URL, category
- Adoption steps
- Expected benefit

## 7. Action Items (Priority Order)
Numbered list of concrete next steps, sorted by impact/effort ratio.
Each item has:
- [ ] Description
- Effort: [5 min | 30 min | 2 hours | 1 day]
- Impact: [high | medium | low]
- Dependency: [none | requires X]
```

### Step 4: Update the tools registry

For each new repo that passes quality filtering (relevance >= 7/10, stars >= 100 or exceptionally relevant):

1. **Add a row to `~/.claude/tools-registry/REGISTRY.md`** with:
   - Repo link
   - Category
   - Stars (approximate)
   - Status: `tracked`
   - Date added
   - Key value (1-line)
   - Link to review file

2. **Create a stub review file** at `~/.claude/tools-registry/reviews/<repo-name>.md`:
   ```markdown
   # Review: <owner>/<repo>
   **URL:** <url>
   **Stars:** <n>
   **Status:** `tracked`
   **Added:** <date>
   **Source:** agentic-radar scan

   ## Summary
   <2-3 sentence description>

   ## Integration Potential
   <How it could improve the user's Claude Code setup>

   ## TODO
   - [ ] Full review
   - [ ] Test integration
   - [ ] Adoption decision
   ```

3. **Update "Last scouted" date** in REGISTRY.md header.

### Step 5: Propose CLAUDE.md changes

Based on the setup audit and discovered patterns, propose specific changes to `~/.claude/CLAUDE.md`.

Rules:
- Categorize each change by priority and impact:
  - **P0 (auto-apply):** Adding new tool/skill references, adding new entries to tables, softening language, fixing typos, adding non-controversial rules
  - **P1 (auto-apply):** New methodology references, updated URLs, expanded search queries, new MCP server configs
  - **HIGH-IMPACT (flag for user review, do NOT auto-apply):** Changes to Section 1 (Execution Style), modifying permissions in settings.json, security-related changes, removing existing rules, changing default behaviors
- Explain the rationale for each change
- Categorize: [new rule | updated rule | new skill entry | new tool entry | methodology update]

### Step 6: Auto-Investigate & Adopt

This step executes adoption automatically after synthesis.

**6a. Auto-apply P0/P1 CLAUDE.md changes:**
For each P0/P1 proposed change from Step 5:
1. Apply the edit to `~/.claude/CLAUDE.md` using the Edit tool
2. Log the change (what was changed, why, source)

For each HIGH-IMPACT change:
1. Print the diff and rationale to the user
2. Print: "HIGH-IMPACT — requires your confirmation. Apply? [y/n]"
3. Do NOT apply until the user confirms

**6b. Investigate standout repos:**
For repos marked "adopt now" in the synthesis report, launch investigation agents (`run_in_background: true`, `mode: bypassPermissions`) to:
1. Clone or fetch the repo's key files (README, config examples, core source)
2. Identify patterns, configs, or skills that can be cherry-picked into `~/.claude/`
3. If the repo provides a skill, hook, or script: copy it into the appropriate `~/.claude/` subdirectory
4. If the repo provides a CLAUDE.md pattern: extract the useful parts and merge them as P0/P1 changes

**6c. Commit all adopted changes:**
```bash
cd ~/.claude && git add -A && git commit -m "agentic-radar: auto-adopt P0/P1 changes from {DATE} scan" && git push
```

Update `~/.claude/CHANGELOG.md` with what was adopted.

**6d. Summary to user:**
Print a table:
| Change | Priority | Status |
|--------|----------|--------|
| ... | P0 | Applied |
| ... | HIGH-IMPACT | Awaiting confirmation |

### Step 7: Report completion

When everything is done, tell the user:
- Report saved to `{REPORT_PATH}`
- N new repos added to registry
- N stub reviews created
- N P0/P1 changes auto-applied, N HIGH-IMPACT changes awaiting confirmation
- N standout repos investigated, N patterns adopted
- Top recommended action (the single highest-impact improvement)

### Step 8: Commit registry changes

```bash
cd ~/.claude && git add tools-registry/ && git commit -m "tools-registry: agentic-radar scan {DATE}, added N repos" && git push
```

Update `~/.claude/CHANGELOG.md` with a one-line entry.

---

## Search Queries Reference

### GitHub Searches
- `claude-code` (topic, sort: stars)
- `CLAUDE.md template` OR `CLAUDE.md config` (sort: updated)
- `mcp server claude` OR `mcp server anthropic` (sort: stars, pushed: last 30 days)
- `claude hooks` OR `claude-code hooks` (sort: updated)
- `agentic workflow` LLM (sort: stars, pushed: last 3 months)
- `coding agent framework` (current year) (sort: stars)
- `claude subagent` OR `claude background agent`
- `awesome-claude` OR `awesome-mcp` OR `awesome-agents`
- `ai coding assistant` framework (sort: stars)
- `agent orchestration` patterns (sort: stars)

### Community Searches
- `site:news.ycombinator.com "claude code"` (last 30 days)
- `site:reddit.com/r/ClaudeAI CLAUDE.md OR hooks OR skills`
- `"claude code" best practices` (last 30 days)
- `"agentic coding" patterns OR workflow`
- `"MCP server" useful OR recommended`

---

## Quality Filtering Criteria

### Repos to INCLUDE
- Claude Code specific tools, configs, or extensions
- MCP server implementations with clear Claude Code use cases
- Agent orchestration frameworks applicable to Claude Code workflows
- Hooks, skills, or slash command libraries
- Configuration management and dotfile patterns
- Productivity tools that integrate with Claude Code
- Novel agentic patterns documented with implementation

### Repos to EXCLUDE
- Generic LLM wrappers with no Claude specificity
- Repos with <50 stars unless exceptionally relevant
- Jailbreak prompts or system prompt leaks
- Abandoned repos (no commits in 6+ months, no issues activity)
- Repos that duplicate functionality already well-covered in the registry

### Community Posts to INCLUDE
- Concrete tips with implementation details
- Workflow patterns with before/after comparisons
- Tool announcements with demos
- Benchmark results comparing approaches
- Anti-patterns and lessons learned

### Community Posts to EXCLUDE
- Vague opinions without actionable content
- "Claude is amazing/terrible" without specifics
- Feature requests to Anthropic (not actionable for the user)
- Comparisons that are purely about model quality, not tooling

---

## Automation (launchd / cron)

To run weekly, create a launchd plist at `~/Library/LaunchAgents/com.claude.agentic-radar.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude.agentic-radar</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>~/.claude/scripts/agentic-radar.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key>
        <integer>1</integer>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/agentic-radar.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/agentic-radar.err</string>
</dict>
</plist>
```

Corresponding script at `~/.claude/scripts/agentic-radar.sh`:
```bash
#!/bin/bash
claude -p "Run the agentic-radar skill. Today is $(date +%Y-%m-%d)."
```

For Linux, use cron instead:
```cron
0 3 * * 1 ~/.claude/scripts/agentic-radar.sh
```

---

## Differences from claude-tools-scout

| Aspect | claude-tools-scout | agentic-radar |
|--------|-------------------|---------------|
| Scope | Claude Code repos only | Agentic systems + Claude Code + AI agent patterns broadly |
| Agents | 3 parallel agents | 5 parallel agents |
| Self-audit | No | Yes — audits ~/.claude/ setup and proposes improvements |
| Deep analysis | Quick eval (README only) | Deep analysis (README + source + issues) |
| Output | Scouting report (inline) | Full dated report file + registry updates + CLAUDE.md diffs |
| Community | Basic community search | Structured insight extraction with actionability scoring |
| Frequency | Weekly (Monday) | Weekly (Monday, same day as scout) |

The two skills are complementary: scout is a lightweight registry updater; radar is a comprehensive improvement engine.

> **Note:** `claude-tools-scout` is a separate private skill not included in this repo. Agentic-radar works independently without it and provides a superset of scout's functionality.

---

## Notes

- Auto-apply P0/P1 changes to CLAUDE.md. Flag HIGH-IMPACT changes (execution style, permissions, security) for user confirmation.
- The setup audit (Agent 5) is the most unique value-add. Prioritize its findings in the synthesis.
- When the same repo appears in both GitHub trending and community discussions, boost its relevance score.
- Keep reports in a consistent directory for longitudinal tracking — compare week-over-week.
- If a previous radar report exists, Agent 5 should also check which prior recommendations were adopted vs. ignored.
- The registry update step should deduplicate against both the existing registry AND repos found by claude-tools-scout.
- Star thresholds are guidelines, not hard rules. A 50-star repo that perfectly fills a gap in the user's setup is worth tracking.
