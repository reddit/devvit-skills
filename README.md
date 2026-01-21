# Agent Skills

A collection of skills for AI coding agents focused on building, maintaining, and debugging [Devvit applications](https://developers.reddit.com/).

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Installation

```bash
npx add-skill reddit/devvit-skills
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

### devvit-logs

Stream logs for an installed Devvit app for quick debugging. Wraps `devvit logs` and captures a short window of output.

**Use when:**
- "devvit logs for r/my-subreddit"
- "stream logs for my app"
- "check logs for this install"
- "show logs since 1h"

**Examples:**
```
devvit logs for r/my-subreddit
```
```
stream logs for r/my-subreddit my-app since 1h
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