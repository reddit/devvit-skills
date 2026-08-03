# Devvit Skills

> All skills are experimental and subject to change at any time!

A collection of skills for AI coding agents focused on building, maintaining, and debugging [Devvit applications](https://developers.reddit.com/).

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Installation

```bash
npx skills add reddit/devvit-skills
```

Then, copy and paste this rule into your project's `AGENTS.md`:

```md
Always use the devvit-docs skill when I need library or API documentation for Devvit, including code generation and setup or configuration steps, without requiring me to explicitly ask. This includes Devvit capabilities such as Redis, the Reddit API, Media, and more, because Devvit has specific rules and documentation.
```

## Available Skills

### devvit-docs

Look up Devvit documentation from `reddit/devvit-docs` and answer questions with citations from that repo.

**Use when:**

- "How do I configure devvit.json?"
- "Show me an example of a menu action"
- "What triggers are available in Devvit?"
- "Where is the form API documented?"

**Examples:**

```
Explain how to add a custom post action in Devvit
```

```
Find the docs for scheduled triggers in Devvit
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**

```
Check logs for r/devvit since 30m
```

```
Where is the post composer documented in Devvit?
```
