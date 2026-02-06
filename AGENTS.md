# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository (Opencode, Codex, Claude, Copilot, etc.).

## Repository Overview

A skills store for [Devvit](https://developers.reddit.com/) development. Skills are packaged instructions and scripts that extend AI agent capabilities. End users install them via:

```bash
npx skills add reddit/devvit-skills
```

This copies (or symlinks) each skill directory into the consumer's agent-specific location (e.g., `.cursor/skills/devvit-docs/` for Cursor, `.claude/skills/devvit-docs/` for Claude Code). The agent discovers skills by scanning those directories and reading the `SKILL.md` frontmatter.

Skills follow the [Agent Skills specification](https://agentskills.io).

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/             # kebab-case directory name
    SKILL.md                # Required: skill definition
    scripts/                # Optional: executable scripts
      {script-name}.cjs     # Node.js scripts (CommonJS)
```

When installed, the entire `{skill-name}/` directory is copied into the consumer's project. Everything inside it (scripts, data files, etc.) travels together and is accessible relative to the SKILL.md.

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `devvit-deploy`, `log-monitor`)
- **SKILL.md**: Always uppercase, always this exact filename
- **Scripts**: `kebab-case.cjs` (e.g., `deploy.cjs`, `fetch-logs.cjs`). Use `.cjs` so scripts work in consumer projects that have `"type": "module"`.

### SKILL.md Format

````markdown
---
name: {skill-name}
description: One sentence describing when to use this skill. Include trigger phrases.
---

# {Skill Title}

{Brief description of what the skill does.}

## How It Works

{Numbered list explaining the skill's workflow}

## Usage

```bash
node ./scripts/{script}.cjs [args]
```

Script paths are relative to this skill's directory.

**Arguments:**

- `arg1` - Description (defaults to X)

**Examples:**

```bash
node ./scripts/{script}.js example-arg
```

## Output

{Example output the agent will see}

## Present Results to User

{Template for how the agent should format results when presenting to users}

## Troubleshooting

{Common issues and solutions}
````

### Best Practices for Context Efficiency

Skills are loaded on-demand — only the skill name and description are loaded at startup. The full `SKILL.md` loads into context only when the agent decides the skill is relevant. To minimize context usage:

- **Keep SKILL.md under 500 lines** — put detailed reference material in separate files
- **Write specific descriptions** — helps the agent know exactly when to activate the skill
- **Use progressive disclosure** — reference supporting files that get read only when needed
- **Prefer scripts over inline code** — script execution doesn't consume context (only output does)
- **File references work one level deep** — link directly from SKILL.md to supporting files

### Script Requirements

All skills must work on **Windows, Linux, and macOS**. Prefer cross-platform Node.js scripts and avoid shell-specific constructs (no `bash -c`, no `/bin/sh`, no PowerShell).

- Use Node.js scripts with `.cjs` extension (CommonJS — works regardless of consumer's `"type"` field)
- Use `process.stderr.write(...)` for status messages
- Write machine-readable output (JSON) to stdout
- Use `path.join()` for all file paths — never hardcode `/` or `\`
- Ensure cleanup of temp files/directories
- Script paths are relative to the skill directory (e.g., `./scripts/my-script.cjs`)
