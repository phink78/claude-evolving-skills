---
name: codex-agent
description: >
  Delegates a task to OpenAI Codex CLI as a subagent and returns its response.
  Use this skill whenever the user wants Codex's opinion, a second opinion from
  Codex, wants to ask Codex to do something, wants to compare Claude vs Codex
  on a task, or says anything like "ask Codex", "have Codex look at this",
  "what does Codex think", "run this through Codex", or "get Codex to review".
  Also use when the user wants multiple AI perspectives on code, writing, or analysis.
---

# Codex Agent Skill

Runs a prompt through the Codex CLI non-interactively and returns the full response.

## Models

| Model | Flag | Use case |
|-------|------|----------|
| **gpt-5.4** (default) | `-c model="gpt-5.4"` | 1M context, computer use, tool search. Best for complex analysis, long files, multi-step reasoning. |
| **gpt-5.4-mini** | `-c model="gpt-5.4-mini"` | 30% of full-model limits. Good for simple delegations, quick opinions, short reviews. |

> **Note:** Model names change frequently. Update the table above when new models are released.
> Check `codex --help` or the vendor-docs-radar reports for the latest.

Use `model_reasoning_effort` to control cost/latency tradeoff:
- `-c model_reasoning_effort="high"` — complex tasks (architecture review, debugging, research)
- `-c model_reasoning_effort="low"` — simple tasks (formatting, renaming, quick checks)

## How to invoke

Use `codex exec` for non-interactive (headless) mode:

```bash
codex exec "<prompt>"
```

Or pipe a long prompt via stdin:

```bash
echo "<prompt>" | codex exec -
```

Or combine file content with a prompt:

```bash
cat <file> | codex exec - --config 'approval_policy="auto-edit"'
```

## Key flags

- `codex exec "<prompt>"` — run non-interactively, prints response to stdout
- `-c model="gpt-5.4"` — use default model (1M context)
- `-c model="gpt-5.4-mini"` — use fast/cheap model for simple tasks
- `-c model_reasoning_effort="high"` — max reasoning for complex tasks
- `-c 'sandbox_permissions=["disk-full-read-access"]'` — allow file reads in sandbox

## Structured output parsing

Use the `phase` parameter to get structured output from Codex, useful for parsing responses programmatically:

```bash
codex exec "<prompt>" -c phase="plan"    # returns a structured plan
codex exec "<prompt>" -c phase="execute" # returns structured execution steps
```

## Prompt Structure

Use the four-element structure (OpenAI best practice) when building prompts for Codex:

```
Goal: <what the task should accomplish>
Context: <relevant files, project state, background>
Constraints: <what NOT to do, boundaries, style rules>
Done-when: <clear completion criteria>
```

**Example:**

```bash
codex exec "Goal: Find race conditions in the connection pool.
Context: $(cat src/pool.py)
Constraints: Do not suggest rewriting in async. Focus on threading bugs only.
Done-when: List each race condition with line number and a one-line fix."
```

For simple one-shot questions (quick opinions, short reviews), a plain prompt is fine — reserve the four-element structure for tasks where precision matters.

## Workflow

1. Formulate the prompt using the four-element structure above — include all context Codex needs (paste file contents, code snippets, etc. directly into the prompt since Codex won't have access to local files unless you pass them)
2. Choose model: `gpt-5.4` for complex tasks, `gpt-5.4-mini` for simple ones
3. Run via Bash tool: `codex exec "<prompt>"`
4. Capture stdout as the response
5. Return Codex's full response to the user, clearly attributed as "Codex says:"

## Passing file content

When reviewing a file, read it first and include its contents in the prompt:

```bash
codex exec "Review this code for bugs:\n\n$(cat path/to/file.py)"
```

Or for long files, use stdin:

```bash
cat path/to/file | codex exec "Review this code for bugs:\n\n-"
```

## Notes

- Codex runs as a sandboxed agent — it may take 30-120 seconds for complex tasks
- For very long prompts (e.g., full papers), pipe via stdin rather than inline
- The response comes from OpenAI's models; clearly label it as coming from Codex when presenting to the user
- `codex exec` is the preferred mode for non-interactive automation (no TTY needed, CI-friendly)
- gpt-5.4 supports 1M context — no need to truncate large files or codebases

## Prerequisites

- Install Codex CLI: `npm install -g @openai/codex` or follow https://github.com/openai/codex
- Authenticate: set `OPENAI_API_KEY` environment variable
