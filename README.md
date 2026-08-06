# Devvit skills

Give your AI coding agent better instructions for building, maintaining, and debugging [Devvit apps](https://developers.reddit.com/).

> These skills are experimental and may change.

## What is a skill?

A skill is a small set of instructions and scripts that teaches an AI coding agent how to handle a specific kind of task. It is not a dependency in your Devvit app, and it does not become part of your production code.

After you install a skill, you can talk to your agent as usual. The agent reads the skill's description and uses it when your request matches. For example, a documentation skill can teach the agent where to find the right Devvit docs and how to cite them.

This repository uses the [`vercel-labs/skills`](https://github.com/vercel-labs/skills) installer and follows the [Agent Skills](https://agentskills.io/) format.

## Install

You need Node.js and a [supported coding agent](https://github.com/vercel-labs/skills#supported-agents).

Open a terminal in your Devvit project, then run:

```bash
npx skills add reddit/devvit-skills
```

The installer will ask which skills and coding agents you want to use. If you are unsure, select `devvit-docs`, choose your coding agent, and use the recommended symlink installation.

If your agent does not notice the newly installed skill, start a new session and try again.

Then ask a normal Devvit question:

```text
How do I add a custom post action to my Devvit app?
```

Your agent should recognize that the question matches `devvit-docs` and use the skill automatically.

## Install only what you need

As this collection grows, you can inspect the available skills before installing anything:

```bash
npx skills add reddit/devvit-skills --list
```

Install one skill by name:

```bash
npx skills add reddit/devvit-skills --skill devvit-docs
```

The default installation applies to the current project. Add `--global` if you want the skill available in all of your projects:

```bash
npx skills add reddit/devvit-skills --skill devvit-docs --global
```

## Do I need to edit `AGENTS.md`?

No. Each skill already tells your agent when to use it.

If your team wants an extra reminder in its agent instructions, you can add this optional, skill-independent rule:

```md
Use installed Devvit skills when their descriptions match the task.
```

This rule applies only to the Devvit skills a developer chooses to install, including skills added to this repository in the future.

## Available skills

### `devvit-docs`

Recommended for Devvit development. This skill searches the official [`reddit/devvit-docs`](https://github.com/reddit/devvit-docs) repository and cites the files and sections used in its answer. It also looks for the Devvit version in your project so it can prefer matching documentation.

Use it for questions such as:

```text
How do I configure devvit.json?
```

```text
Show me an example of a menu action.
```

```text
What triggers are available in Devvit?
```

```text
Where is the form API documented?
```

The first lookup downloads a local, text-only copy of the docs. Later lookups reuse that cache and refresh it when needed. The skill requires Git and network access for the initial download.

## Manage installed skills

The `skills` CLI can show, update, and remove installed skills:

```bash
npx skills list
npx skills update devvit-docs
npx skills remove devvit-docs
```

See the [`vercel-labs/skills` README](https://github.com/vercel-labs/skills) for agent-specific installation options and the full CLI reference.
