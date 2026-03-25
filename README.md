# Codex-Driven Development

A Claude Code plugin that uses Claude as an orchestrator to plan work and dispatch [OpenAI Codex CLI](https://github.com/openai/codex) agents to write the actual code.

## Install

### Step 1: Add the marketplace

```
/plugin marketplace add tarundevi/codex-driven-development
```

### Step 2: Install the plugin

```
/plugin install codex-driven-development
```

### Step 3: Reload plugins

```
/reload-plugins
```

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured
- [Codex CLI](https://github.com/openai/codex) installed and configured (`npm install -g @openai/codex`)

## How It Works

| Phase | Who | What |
|-------|-----|------|
| Planning | Claude | Writes full implementation plan with bite-sized tasks |
| Rules Injection | Claude | Reads CLAUDE.md files, extracts relevant coding rules for Codex prompts |
| Dispatch | Claude | Selects model/reasoning per task complexity, runs `codex exec` |
| Implement | Codex | Writes code and tests — no git commands, structured report output |
| Verify | Claude | Parses report, runs tests independently, checks git diff |
| Spec Review | Codex | Verifies implementation matches spec (PASS/FAIL) |
| Quality Review | Codex | Verifies code quality (PASS/FAIL) |
| Retry | Claude | Re-dispatches with feedback, escalates model if needed |
| Commit | User | All git commits are manual |

### Model Selection

| Task Type | Model | Reasoning |
|-----------|-------|-----------|
| Mechanical (1-2 files, clear spec) | `gpt-5.4-mini` | `low` |
| Standard (multi-file, moderate judgment) | `gpt-5.4` | `medium` |
| Complex (architectural, cross-cutting) | `gpt-5.4` | `high` |
| Review | `gpt-5.4` | `medium` |

### Retry & Escalation

Failed review → retry up to 3x at same model → escalate model → ask user.

## Usage

The skill shows up in your skills list once installed. You can invoke it with:

```
Use codex-driven-development to implement this feature
```

Or reference it when asking Claude to build something:

```
Plan and build X using Codex agents to write the code
```

## Updating

```
/plugin marketplace update codex-driven-development
```
