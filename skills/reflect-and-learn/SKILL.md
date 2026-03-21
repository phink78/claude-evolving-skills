---
name: reflect-and-learn
description: |
  Weekly self-reflection skill that reviews past sessions, identifies workflow weaknesses,
  debates improvements with Gemini/Codex, scores proposals via rubrics, retrospects on
  past adoptions, and auto-evolves CLAUDE.md and skill configurations.
  Use for "reflect", "self-improve", "review my workflows", "what should we improve",
  "run reflection", "evolve config", or weekly automated cron.
---

# Reflect and Learn (v2)

A self-evolving meta-improvement loop drawing from:
- **AFlow** [Zhang et al., ICLR 2025](https://arxiv.org/abs/2410.10762): tree-structured config history, soft mixed selection, error logs as optimizer context
- **AgentEvolver** [Zhai et al., 2025](https://arxiv.org/abs/2511.10395): dual-channel scoring (process+outcome), experience stripping for rule pruning, 50/50 exploration/exploitation
- **Live-SWE-agent** [Xia et al., 2025](https://arxiv.org/abs/2511.13646): step-level reflection injection, ephemeral→persistent tool promotion
- **EvoAgentX/SEW** [Wang et al., 2025](https://arxiv.org/abs/2507.03616): two-level evolution (structure then prompts), hyper-evolution of the mutator
- **Self-Evolving Agents Survey** [Fang et al., 2025](https://arxiv.org/abs/2508.07407): memory consolidation, text gradients, longitudinal evaluation, tool co-evolution

## When to Use

- Weekly automated cron (Wednesday 3:00am)
- User says "reflect", "self-improve", "review workflows", "evolve config"
- After a particularly rough session where multiple things went wrong
- When user asks "what should we improve?"

## Output

- Dated reflection report: `~/.claude/evolution-history/reflections/YYYY-MM-DD-reflection.md`
- Updated `~/.claude/evolution-history/evolution-tree.jsonl` (AFlow-style tree-structured history)
- CLAUDE.md diffs (auto-applied P0/P1, flagged HIGH-IMPACT)
- Memory consolidation results (pruned stale entries, promoted recurring patterns)
- Git commit of all changes

---

## Execution Instructions

### Step 0: Setup

```
DATE=$(date +%Y-%m-%d)
WORK_DIR=$(mktemp -d)
REFLECTION_DIR=~/.claude/evolution-history/reflections
SCOREBOARD=~/.claude/evolution-history/scoreboard.jsonl
mkdir -p "$REFLECTION_DIR"
```

### Step 1: Extract Past Week's Sessions

Find all conversation JSONL files modified in the last 7 days:

```bash
find ~/.claude/projects -name "*.jsonl" -mtime -7 -type f | head -50
```

For each session file, extract a compressed summary using `jq`:
- User prompts (type=user → .message.content)
- Assistant tool usage patterns (type=assistant → tool_use entries)
- Error messages and retries
- Token usage (.message.usage)
- Session duration (first timestamp to last timestamp)

Write per-session summaries to `{WORK_DIR}/sessions/`. Each summary should capture:
- **Task type**: code generation, debugging, research, config, refactoring
- **Outcome**: success, partial, failure
- **Friction points**: retries, user corrections, permission denials, tool errors
- **Efficiency**: token count, tool call count, elapsed time
- **User feedback signals**: corrections ("no", "don't", "stop"), confirmations ("yes", "perfect", "exactly")

### Step 2: Launch 6 Parallel Analysis Agents

Fire ALL agents simultaneously with `run_in_background: true`, `mode: bypassPermissions`.

**Agent 1 — Failure Analyst + Text Gradient Generator** (inspired by [TextGrad, Yuksekgonul et al. 2024](https://arxiv.org/abs/2406.07496))
```
Review the session summaries in {WORK_DIR}/sessions/.
Focus ONLY on sessions with failures, retries, or user corrections.

For each failure pattern:
1. What went wrong (specific tool/approach/assumption)
2. How many sessions exhibited this pattern
3. Root cause (missing rule? wrong default? knowledge gap?)
4. **Text gradient**: trace the failure back through the config chain:
   "The output was wrong BECAUSE [tool X was chosen] BECAUSE [CLAUDE.md rule Y says Z]
   BECAUSE [the rule was written for context W which no longer applies]."
   Generate the inverse edit: the specific CLAUDE.md diff that would prevent this failure.
5. **Error log context**: for each failure, include the concrete input, expected output,
   actual output, and error message — these feed directly into the optimizer in Step 3.

Output: {WORK_DIR}/analysis-failures.md
Format: numbered list of failure patterns with frequency, root cause, text gradient, and proposed diff.
```

**Agent 2 — Efficiency Auditor**
```
Review the session summaries in {WORK_DIR}/sessions/.
Analyze efficiency metrics across all sessions:

1. Average tool calls per task type — are any task types using excessive calls?
2. Token usage patterns — which patterns consume the most tokens?
3. Time-to-completion — which task types are slowest?
4. Parallelization usage — are agents being used effectively?
5. Tool selection — any cases where a better tool existed?

Compare against previous week's metrics if available in the scoreboard.

Output: {WORK_DIR}/analysis-efficiency.md
Format: metrics table + specific improvement proposals with expected impact.
```

**Agent 3 — User Satisfaction Detector**
```
Review the session summaries in {WORK_DIR}/sessions/.
Focus on signals of user satisfaction and dissatisfaction:

1. Explicit corrections: count and categorize ("don't do X", "not that", "wrong")
2. Explicit praise: count and categorize ("perfect", "exactly", "great")
3. Repeated instructions: things the user had to say more than once
4. Abandoned tasks: sessions where user stopped mid-task
5. Flow breakers: moments where the user had to re-explain or redirect

Extract feedback patterns that should become persistent rules.

Output: {WORK_DIR}/analysis-satisfaction.md
Format: satisfaction signals table + proposed feedback memories + proposed CLAUDE.md rules.
```

**Agent 4 — Retrospective Evaluator**
```
Review the evolution history:
- Read ~/.claude/evolution-history/scoreboard.jsonl for past config changes and their scores
- Read the last 2-3 reflection reports in ~/.claude/evolution-history/reflections/
- Read the current CLAUDE.md

For each change adopted in the past 2-4 weeks:
1. What was the change?
2. What was the expected improvement?
3. What evidence exists that it helped or hurt? (look at session data)
4. Score: CONFIRMED_HELPFUL | INCONCLUSIVE | CONFIRMED_HARMFUL | NEEDS_MORE_DATA

Flag any CONFIRMED_HARMFUL changes for rollback.

Output: {WORK_DIR}/analysis-retrospective.md
Format: change-by-change evaluation with evidence and verdict.
```

**Agent 5 — Meta-Evolution Analyst (Hyper-Evolution, inspired by [SEW, Liu et al. 2025](https://arxiv.org/abs/2505.18646))**
```
Review the current CLAUDE.md self-evolution protocol (Section 6).
Review the reflection skill itself (this SKILL.md).
Review past reflection reports and the evolution tree.

PART A — Process Self-Assessment:
1. Is the reflection process itself improving over time?
2. Are we catching the right things? Missing anything?
3. Is the rubric correctly weighted? (check: do adopted changes with high scores
   actually get CONFIRMED_HELPFUL more often than low-scored ones?)
4. Should the reflection frequency change?

PART B — Hyper-Evolution (evolve the mutator):
Instead of just proposing CLAUDE.md changes, propose changes to HOW we propose changes:
1. Should the analysis agents' prompts be modified?
2. Should the scoring weights be rebalanced based on historical accuracy?
3. Should the adoption thresholds (P0 >= 8.0, P1 >= 7.0) be adjusted?
4. Should we add/remove analysis agents?
Generate a "mutation strategy" — a meta-prompt that describes the current best approach
to generating improvements, based on what has worked in past reflections.

Output: {WORK_DIR}/analysis-meta.md
Format: meta-evaluation + proposed process improvements + updated mutation strategy.
```

**Agent 6 — Tool & Pattern Co-Evolution Detector** (inspired by [Live-SWE-agent, Xia et al. 2025](https://arxiv.org/abs/2511.13646) + [Survey, Fang et al. 2025](https://arxiv.org/abs/2508.07407))
```
Review the session summaries in {WORK_DIR}/sessions/.
Focus on detecting emergent tool patterns and repeated ad-hoc constructions:

1. **Repeated command patterns**: shell commands or tool sequences used 3+ times
   across sessions that aren't captured as skills or aliases
2. **Ad-hoc scripts**: temporary scripts created during sessions that could be
   promoted to persistent skills (ephemeral→persistent pipeline)
3. **Missing tool categories**: using Live-SWE-agent's category audit —
   do we have good coverage for: edit, view, search, domain-specific analysis,
   multi-file operations, diffing? Which category has the most session failures?
4. **Skill usage patterns**: which skills are used most/least? Any skills never triggered?
5. **MCP server gaps**: tasks where an MCP server would have helped but didn't exist

For each detected pattern, propose:
- A new skill, alias, or MCP server configuration
- Estimated effort: [5 min | 30 min | 2 hours]

Output: {WORK_DIR}/analysis-tools.md
Format: detected patterns table + proposed new tools/skills.
```

### Step 3: Multi-Voice Debate

After all 6 agents complete, launch a synthesis + debate phase:

**Synthesis Agent (background, bypassPermissions):**
```
Read all 6 analysis files from {WORK_DIR}/analysis-*.md.

PART A — SYNTHESIS:
Cross-reference findings. Group into:
- CONFIRMED (2+ agents agree): high-confidence improvements
- SINGLE_SIGNAL (1 agent only): needs validation
- CONTRADICTORY (agents disagree): needs debate

PART B — DUAL-CHANNEL SCORING (inspired by AgentEvolver's dual-channel attribution):
Score each proposed change on TWO independent channels, normalized separately:

Channel 1 — PROCESS QUALITY (was the reasoning clean?):
| Dimension | Weight | Description |
|-----------|--------|-------------|
| Evidence | 3x | How strong is the evidence from session data? |
| Generality | 2x | Applies broadly vs. narrow edge case? |
| Simplicity | 1x | Easy to implement and understand? |

Channel 2 — OUTCOME QUALITY (will it actually help?):
| Dimension | Weight | Description |
|-----------|--------|-------------|
| Impact | 3x | How much will this improve daily workflow? |
| Safety | 2x | Risk of regression or unintended side effects? (10=safe) |
| Text Gradient Strength | 1x | How clear is the causal chain from root cause to fix? |

Each channel scored 1-10 independently, then combined:
Composite = 0.5 * ProcessScore + 0.5 * OutcomeScore

PART C — TREE-STRUCTURED SELECTION (inspired by AFlow):
Use soft mixed probability to select which proposals to advance:
- 40% uniform random across all proposals (exploration)
- 60% softmax-weighted by composite score (exploitation)
This prevents always refining the top-scored proposal and missing better alternatives.
Select top 5 candidates.

PART D — ERROR LOG CONTEXT:
For each candidate, attach the concrete failure examples from Agent 1's error logs.
These feed into the Gemini/Codex debate so external models can see the actual failures,
not just our interpretation of them.

Output: {WORK_DIR}/synthesis.md
```

**Then launch Gemini + Codex in parallel** using the `gemini-agent` and `codex-agent` skills:

Prompt for both:
```
I'm reviewing my Claude Code configuration for weekly self-improvement.
Here are the top 5 proposed changes from session analysis:

[paste top 5 from synthesis]

For each proposal:
1. Do you agree this is a real problem worth fixing?
2. Is the proposed solution correct, or do you suggest an alternative?
3. What could go wrong if we adopt this?
4. Score 1-10 on impact and 1-10 on risk.

Also: are there any improvements you'd suggest that aren't in this list?
```

### Step 4: Final Decision & Adoption

After Gemini + Codex respond, make final decisions:

**Classification:**
- **P0-AUTO** (score >= 8.0, all 3 voices agree, Safety >= 8): Apply immediately
- **P1-AUTO** (score >= 7.0, 2/3 voices agree, Safety >= 7): Apply immediately
- **HIGH-IMPACT** (score >= 6.0 but Safety < 7 or voices disagree): Flag for user
- **DEFER** (score < 6.0): Log but don't apply
- **ROLLBACK** (retrospective flagged CONFIRMED_HARMFUL): Revert immediately

**For each adopted change:**
1. Edit the target file (CLAUDE.md, skill files, scripts, etc.)
2. Log to scoreboard:
   ```jsonl
   {"date": "YYYY-MM-DD", "change_id": "reflect-YYYYMMDD-N", "description": "...", "score": N.N, "voices": {"claude": N, "gemini": N, "codex": N}, "status": "adopted", "expected_impact": "..."}
   ```

**For ROLLBACK items:**
1. Revert the change in target file
2. Update scoreboard entry: `"status": "rolled_back", "reason": "..."`

### Step 4b: Memory Consolidation (inspired by [MemoryBank, Zhong et al. 2024] via Survey)

After adoption decisions, run a memory consolidation pass:

1. **Read all memory files** in `~/.claude/projects/*/memory/`
2. **Score each memory** by: access frequency (how often referenced in sessions),
   recency (last relevant session), and redundancy (does another memory cover this?)
3. **Prune**: remove memories not referenced in any session for 4+ weeks
4. **Consolidate**: merge memories that cover the same topic into a single entry
5. **Promote**: recurring feedback patterns (3+ sessions) that aren't yet in CLAUDE.md
   should be proposed as new rules

### Step 4c: Experience Stripping — Rule Necessity Test (inspired by [AgentEvolver, Zhai et al. 2025])

For CLAUDE.md rules that have been in place for 4+ weeks:

1. **Identify candidate rules** for stripping test (rules added by past reflections)
2. **Check session data**: are there recent sessions where the rule was actively needed
   (i.e., would have failed without it)?
3. **If no evidence of necessity in 4 weeks**: mark as STRIP_CANDIDATE
4. **Propose removal** of STRIP_CANDIDATEs with Safety score override (flag as HIGH-IMPACT
   so user confirms — we never auto-remove rules)

This prevents config bloat. Rules the agent has "internalized" or that addressed
one-time issues get pruned, keeping CLAUDE.md lean.

### Step 4d: Evolution Tree Update (inspired by [AFlow, Zhang et al. 2025])

Update the tree-structured evolution history at `~/.claude/evolution-history/evolution-tree.jsonl`:

```jsonl
{
  "node_id": "reflect-YYYYMMDD",
  "parent_id": "reflect-YYYYMMDD-prev",
  "date": "YYYY-MM-DD",
  "config_snapshot_hash": "<sha256 of CLAUDE.md>",
  "changes": [
    {"change_id": "...", "description": "...", "status": "adopted|deferred|rolled_back"}
  ],
  "metrics": {
    "sessions_analyzed": N,
    "success_rate": 0.XX,
    "avg_tool_calls": N,
    "avg_tokens": N,
    "user_corrections": N,
    "user_confirmations": N
  },
  "process_score": N.N,
  "outcome_score": N.N,
  "composite_score": N.N,
  "success_branches": ["change_ids that improved metrics"],
  "failure_branches": ["change_ids that degraded metrics"]
}
```

This tree structure allows future reflections to:
- See which modification paths led to improvements vs degradation
- Avoid repeating failed modifications from the same parent state
- Identify high-performing branches to build on

### Step 5: Write Reflection Report

Write the full report to `~/.claude/evolution-history/reflections/{DATE}-reflection.md`:

```markdown
# Weekly Reflection — {DATE}

## Executive Summary
- Sessions analyzed: N
- Failure patterns found: N
- Improvements proposed: N
- Changes adopted: N (P0: X, P1: Y)
- Changes deferred: N
- Rollbacks: N

## Session Statistics
| Metric | This Week | Last Week | Delta |
|--------|-----------|-----------|-------|
| Total sessions | | | |
| Success rate | | | |
| Avg tool calls/task | | | |
| Avg tokens/task | | | |
| User corrections | | | |
| User confirmations | | | |

## Failure Analysis
[From Agent 1]

## Efficiency Analysis
[From Agent 2]

## User Satisfaction
[From Agent 3]

## Retrospective on Past Changes
[From Agent 4]

## Meta-Evolution Notes
[From Agent 5]

## Multi-Voice Debate Summary
| Proposal | Claude | Gemini | Codex | Consensus | Decision |
|----------|--------|--------|-------|-----------|----------|

## Changes Adopted
[Diffs applied with rationale]

## Changes Deferred
[With reasons]

## Optimization Trajectory
[OPRO-style: show last 5 weeks' scores and trend]
```

### Step 6: Commit & Clean Up

```bash
cd ~/.claude
git add -A
git commit -m "reflect-and-learn: weekly reflection {DATE}"
git push
```

Clean up `{WORK_DIR}`.

Report to user:
- Path to reflection report
- Number of changes adopted vs deferred
- Top improvement adopted (1 sentence)
- Trend: improving / stable / degrading

---

## Rubric Reference (Dual-Channel)

### Channel 1 — Process Quality
| Dimension | Weight | 1-3 (Low) | 4-6 (Medium) | 7-10 (High) |
|-----------|--------|-----------|--------------|--------------|
| **Evidence** | 3x | Anecdotal, 1 session | 2-3 sessions, pattern emerging | 5+ sessions, clear pattern |
| **Generality** | 2x | One specific project/task | One task category | Applies across all work |
| **Simplicity** | 1x | Requires new tooling/complex changes | Moderate CLAUDE.md edit | Simple rule addition/modification |

### Channel 2 — Outcome Quality
| Dimension | Weight | 1-3 (Low) | 4-6 (Medium) | 7-10 (High) |
|-----------|--------|-----------|--------------|--------------|
| **Impact** | 3x | Cosmetic, rare edge case | Noticeable but not daily | Daily workflow improvement |
| **Safety** | 2x | Could break workflows, hard to revert | Minor regression risk | No regression risk, easily reversible |
| **Text Gradient Strength** | 1x | Vague correlation | Plausible causal chain | Clear root-cause→fix with specific CLAUDE.md line |

### Composite Score
`Composite = 0.5 * ProcessScore + 0.5 * OutcomeScore`

Both channels normalized independently (AgentEvolver finding: removing either channel degrades results).

## Evolution Tree Format (evolution-tree.jsonl)

Each line is a tree node representing a complete reflection cycle:
```jsonl
{"node_id": "reflect-YYYYMMDD", "parent_id": "reflect-YYYYMMDD-prev", "date": "YYYY-MM-DD", "config_hash": "abc123", "changes": [{"change_id": "reflect-YYYYMMDD-1", "description": "Add rule: always read file before editing", "composite_score": 8.5, "process_score": 8.0, "outcome_score": 9.0, "voices": {"claude": 9, "gemini": 8, "codex": 7}, "status": "adopted", "expected_impact": "Reduce edit failures by ~30%"}], "metrics": {"sessions": 12, "success_rate": 0.83, "avg_tool_calls": 15, "user_corrections": 3, "user_confirmations": 8}, "success_branches": ["reflect-YYYYMMDD-1"], "failure_branches": [], "retrospective_verdicts": {"reflect-YYYYMMDD-2": "CONFIRMED_HELPFUL", "reflect-YYYYMMDD-3": "INCONCLUSIVE"}}
```

The tree structure enables AFlow-style selection: when deciding what to try next,
load the subtree from the current node to see what worked and failed from this exact state.

## SOTA References

| Paper | Year | Key technique adopted |
|-------|------|----------------------|
| [AFlow](https://arxiv.org/abs/2410.10762) | ICLR 2025 | Tree-structured history, soft mixed selection, error logs as context |
| [AgentEvolver](https://arxiv.org/abs/2511.10395) | 2025 | Dual-channel scoring, experience stripping, exploration/exploitation mix |
| [Live-SWE-agent](https://arxiv.org/abs/2511.13646) | 2025 | Step-level reflection, ephemeral→persistent tool promotion |
| [EvoAgentX/SEW](https://arxiv.org/abs/2507.03616) | 2025 | Two-level evolution, hyper-evolution of the mutator |
| [Self-Evolving Agents Survey](https://arxiv.org/abs/2508.07407) | 2025 | Memory consolidation, text gradients, tool co-evolution, longitudinal eval |
| [TextGrad](https://arxiv.org/abs/2406.07496) | Nature 2024 | Natural-language "backpropagation" for config optimization |
