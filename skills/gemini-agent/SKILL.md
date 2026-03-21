---
name: gemini-agent
description: >
  Delegates a task to Google Gemini CLI as a subagent and returns its response.
  Use this skill whenever the user wants Gemini's opinion, a second opinion from
  Gemini, wants to ask Gemini to do something, wants to compare Claude vs Gemini
  on a task, or says anything like "ask Gemini", "have Gemini look at this",
  "what does Gemini think", "run this through Gemini", or "get Gemini to review".
  Also use when the user wants multiple AI perspectives on code, writing, research,
  or analysis — Gemini has a very large context window useful for long documents.
---

# Gemini Agent Skill

Runs a prompt through the Gemini CLI non-interactively and returns the full response.

## Models

| Purpose | Model | When to use |
|---------|-------|-------------|
| **Default (agentic work)** | `gemini-3.1-pro-preview-customtools` | Multi-step tasks, tool use, code generation, reviews, research |
| **Fallback / fast** | `gemini-3.1-flash-lite-preview` | Simple questions, quick lookups, low-latency needs |

Always pass `-m gemini-3.1-pro-preview-customtools` unless speed matters more than quality,
in which case use `-m gemini-3.1-flash-lite-preview`.

> **Note:** Model names change frequently. Update the table above when new models are released.
> Check `gemini --list-models` or the vendor-docs-radar reports for the latest.

## How to invoke

Use `-p` / `--prompt` for non-interactive (headless) mode:

```bash
gemini -m gemini-3.1-pro-preview-customtools -p "<prompt>"
```

Or pipe content via stdin with the prompt appended:

```bash
cat <file> | gemini -m gemini-3.1-pro-preview-customtools -p "<instruction>"
```

## Key flags

- `-p "<prompt>"` — run non-interactively, prints response to stdout
- `-m <model>` — override model
- `-y` / `--yolo` — auto-approve all actions (useful for non-interactive batch tasks)
- `--approval-mode yolo` — same as -y, auto-approve everything

## Thinking level

For complex tasks (research synthesis, architecture review, multi-step reasoning),
pass `thinking_level: high` if the CLI or API supports it. This triggers extended
chain-of-thought and improves accuracy on hard problems.

## Workflow

1. Formulate the prompt — include all context Gemini needs. Gemini has a very large context window (1M+ tokens), so you can pipe entire long documents via stdin.
2. Run via Bash tool: `gemini -m gemini-3.1-pro-preview-customtools -p "<prompt>"` or `cat file | gemini -m gemini-3.1-pro-preview-customtools -p "<instruction>"`
3. Capture stdout as the response
4. Return Gemini's full response to the user, clearly attributed as "Gemini says:"

## Passing file content

For short content, inline it:
```bash
gemini -m gemini-3.1-pro-preview-customtools -p "Review this paper abstract:\n\n<content here>"
```

For long documents (papers, codebases), pipe via stdin — Gemini excels at long context:
```bash
cat /path/to/paper.tex | gemini -m gemini-3.1-pro-preview-customtools -p "Give a critical review of this paper. Focus on: methodology, claims, and weaknesses."
```

## Notes

- Gemini runs non-interactively with `-p`; responses stream to stdout
- For very long documents, Gemini's large context window is a key advantage over other CLIs
- The response comes from Google's Gemini models; clearly label it as coming from Gemini when presenting to the user
- May take 30-90 seconds for complex or long-context tasks
- **Deprecated models (do not use):** `gemini-2.0-flash`, `gemini-2.5-pro`, `gemini-3-pro-preview` — these are shut down or end-of-life

## Prerequisites

- Install Gemini CLI: `npm install -g @google/gemini-cli` or follow https://github.com/google-gemini/gemini-cli
- Authenticate: run `gemini` once interactively to complete OAuth or set `GEMINI_API_KEY`
