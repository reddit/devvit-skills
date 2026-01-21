# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository (Opencode, Codex, Claude, Copilot, etc.).

## Repository Overview

A collection of skills for working with Devvit deployments. Skills are packaged instructions and scripts that extend agent capabilities.

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    scripts/              # Required: executable scripts
      {script-name}.js    # Node.js scripts (preferred)
  {skill-name}.zip        # Required: packaged for distribution
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `devvit-deploy`, `log-monitor`)
- **SKILL.md**: Always uppercase, always this exact filename
- **Scripts**: `kebab-case.js` (e.g., `deploy.js`, `fetch-logs.js`)
- **Zip file**: Must match directory name exactly: `{skill-name}.zip`

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing when to use this skill. Include trigger phrases like "Deploy my app", "Check logs", etc.}
---

# {Skill Title}

{Brief description of what the skill does.}

## How It Works

{Numbered list explaining the skill's workflow}

## Usage

```bash
node /mnt/skills/user/{skill-name}/scripts/{script}.js [args]
```

**Arguments:**
- `arg1` - Description (defaults to X)

**Examples:**
{Show 2-3 common usage patterns}

## Output

{Show example output users will see}

## Present Results to User

{Template for how Claude should format results when presenting to users}

## Troubleshooting

{Common issues and solutions, especially network/permissions errors}
```

### Best Practices for Context Efficiency

Skills are loaded on-demand — only the skill name and description are loaded at startup. The full `SKILL.md` loads into context only when the agent decides the skill is relevant. To minimize context usage across agents:

- **Keep SKILL.md under 500 lines** — put detailed reference material in separate files
- **Write specific descriptions** — helps the agent know exactly when to activate the skill
- **Use progressive disclosure** — reference supporting files that get read only when needed
- **Prefer scripts over inline code** — script execution doesn't consume context (only output does)
- **File references work one level deep** — link directly from SKILL.md to supporting files

### Script Requirements

All skills must be **Windows-compatible**. Prefer cross-platform Node.js scripts and avoid shell-specific constructs.

- Use Node.js scripts by default (`.js`)
- Use `process.stderr.write(...)` for status messages
- Write machine-readable output (JSON) to stdout
- Ensure cleanup of temp files/directories
- Reference the script path as `/mnt/skills/user/{skill-name}/scripts/{script}.js`

### End-User Installation

Document these two installation methods for users:

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**Codex/OpenAI CLI:**
```bash
cp -r skills/{skill-name} ~/.codex/skills/
```

**Opencode:**
```bash
cp -r skills/{skill-name} ~/.opencode/skills/
```

**Copilot (Chat/IDE):**
Add the skill to workspace/project knowledge, or paste `SKILL.md` contents into the conversation.

**Claude (Web):**
Add the skill to project knowledge or paste `SKILL.md` contents into the conversation.

If the skill requires network access, instruct users to add required domains in the agent's settings (for example, `claude.ai/settings/capabilities` for Claude Web).