---
name: cdd-config
description: Configure which Codex models and reasoning levels codex-driven-development should use for mechanical, standard, complex, and review tasks
argument-hint: [refresh]
allowed-tools: [Read, Write, AskUserQuestion, Bash]
---

# Configure Codex-Driven Development

Configure the saved model profile for this plugin.

The user invoked this command with: `$ARGUMENTS`

## Goal

Write `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/plugins/codex-driven-development/config.json` so the main `codex-driven-development` skill can load user-preferred models instead of relying on hardcoded defaults.

## Required Inputs

Before asking questions:

1. Read `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/plugins/codex-driven-development/config.json` if it exists.
2. Read `${CODEX_HOME:-$HOME/.codex}/config.toml` if it exists.
3. Run `codex --version`.
4. Run `codex --help`.
5. Run `codex exec --help`.

## Discovery Rules

- Treat the installed Codex CLI as the freshest local source of truth.
- Do not assume the plugin's baked-in model names are current if the local Codex environment suggests otherwise.
- If the user passed `refresh`, refresh discovery metadata even if a config file already exists.
- If CLI inspection does not reveal a better model list, keep using the existing saved choices or the plugin defaults.

## Conversation Flow

Use `AskUserQuestion` to collect the minimum needed input.

### Question 1: Profile mode

- header: `Profile`
- question: `How should codex-driven-development pick models by default?`
- multiSelect: false
- options:
  - `Recommended` - Use the latest locally discoverable Codex-friendly defaults
  - `Custom` - I want to choose the model for each task class
  - `Keep current` - Preserve my existing saved choices if present

### Question 2: Task profiles

Ask only when the user chooses `Custom`, or when there is no existing config and they do not choose `Recommended`.

Collect choices for all four task classes:

- Mechanical tasks
- Standard tasks
- Complex tasks
- Review tasks

For each class, ask for:

- model name
- reasoning level: `low`, `medium`, or `high`

Use these defaults unless the user changes them:

- Mechanical: fastest available small/default coding model, reasoning `low`
- Standard: balanced default coding model, reasoning `medium`
- Complex: strongest available coding model, reasoning `high`
- Review: balanced or strong coding model, reasoning `medium`

If the local Codex config already defines a default model, include it in your recommendation.

## Output File

Write JSON to `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/plugins/codex-driven-development/config.json` with this shape:

```json
{
  "schemaVersion": 1,
  "discovery": {
    "refreshedAt": "ISO-8601 timestamp",
    "codexVersion": "codex version string",
    "source": "installed-codex-cli",
    "notes": "Short note about how defaults were chosen"
  },
  "profiles": {
    "mechanical": { "model": "model-name", "reasoning": "low" },
    "standard": { "model": "model-name", "reasoning": "medium" },
    "complex": { "model": "model-name", "reasoning": "high" },
    "review": { "model": "model-name", "reasoning": "medium" }
  }
}
```

## Resolution Rules

- If `Keep current` is selected and a valid config exists, preserve the existing `profiles` block and only update `discovery`.
- If `Recommended` is selected, choose profiles from the freshest local signals available in this order:
  1. Existing saved config
  2. `${CODEX_HOME:-$HOME/.codex}/config.toml`
  3. Current plugin defaults:
     - Mechanical: `gpt-5.5-mini` + `low`
     - Standard: `gpt-5.5` + `medium`
     - Complex: `gpt-5.5` + `high`
     - Review: `gpt-5.5` + `medium`
- If `Custom` is selected, use exactly what the user chose.
- Keep the JSON compact and valid.

## Final Response

Report:

- where the config was written
- the final profile table
- that future `codex-driven-development` runs should load this config first

