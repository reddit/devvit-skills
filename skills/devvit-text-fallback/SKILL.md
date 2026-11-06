---
name: devvit-text-fallback
description: Set up text fallback content when creating Devvit web app posts. Use when a developer asks to add textFallback to their app or improve a game's text representation for SEO or screen readers on Reddit.
---

# Set up Devvit text fallback

Add meaningful Markdown or plain text content to a Devvit web app's posts using the Reddit API. Text fallback provides readable content in Old Reddit and supports search discovery. It can offer a text alternative, but does not by itself make the interactive app accessible.

Apply changes to post creation code; leave existing posts unchanged.

## Find the app's post creation path

Read the app's `package.json`, `devvit.json`, and existing post creation code before editing. Find every relevant creation path, including menu actions, install triggers, scheduled daily posts, and shared helpers. Identify existing fallback content and where the app gets its public post data.

Use the APIs supported by the installed SDK:

- Use `reddit` from `@devvit/web/server` in server code and extend `reddit.submitCustomPost(...)`.
- Add `textFallback: { text: fallbackText }` to the existing options. This is a post option, not a `devvit.json` setting. If Reddit API permission is missing, merge `"reddit": true` into the existing `permissions` object.

Use textFallback.text for Markdown-formatted or plain text. Pass the string directly to the SDK; no conversion is needed.

For current API details, consult the official [text fallback guide](https://developers.reddit.com/docs/capabilities/server/text_fallback) and [custom post guide](https://developers.reddit.com/docs/capabilities/creating_custom_post). Check the installed SDK types when a signature differs. If `devvit-docs` is installed, it can locate matching documentation; this skill also works without it.

## Write useful public content

Build the fallback from the same public data used to create the post. Give someone who cannot use the interactive experience enough context to understand it:

- Name the game or app and explain what the player does.
- Include concise rules or instructions. For a daily puzzle, include its date, number, or public theme when available.
- Include relevant public post content, such as a puzzle prompt or scoreboard summary, when it is useful outside the interactive view.

Write directly to the reader. Start instructions with the action they can take, using concrete verbs that fit the app, such as “Browse,” “Choose,” “Find,” or “Play.” Use the app's actual behavior and control labels to decide what to say; do not apply the same call to action to every app.

Keep the focus on the content and task. Omit introductions such as “In the interactive view,” “This app lets you,” or “Use the interface to.” Include the context needed to understand the action, but avoid explaining what the layout or a control's label already makes clear. Keep the fallback understandable on its own by naming its subject and purpose.

Use actual app details rather than generic copy such as “This is an interactive post.” Keep the text readable on its own, with short paragraphs and simple Markdown. The documented limit is 40,000 characters; aim for a concise description rather than filling the limit. Escape dynamic values that should appear as literal text if they can contain Markdown syntax.

Fallback content is shared with everyone. Exclude private state, per-user progress, unrevealed answers, and puzzle solutions. A text fallback provides a readable representation; it does not make an interactive game playable while logged out or replace accessibility work inside the app. Social sharing images and feed metadata are separate tasks.

## Include the fallback when creating posts

Adapt this example to the app's existing handler and entrypoint. Replace the sample game copy with accurate content. Preserve existing options such as `postData`, `styles`, `runAs`, and `userGeneratedContent`.

```ts
import { context, reddit } from "@devvit/web/server";

const fallbackText = [
  "# Daily Word Trail",
  "Find a path from the starting word to the target word by changing one letter at a time.",
  "Each step must be a valid word. Try to reach the target in as few steps as possible.",
  "A new puzzle is published each day. Play today’s puzzle on Reddit.",
].join("\n\n");

const post = await reddit.submitCustomPost({
  subredditName: context.subredditName,
  title: "Daily Word Trail",
  entry: "default",
  textFallback: { text: fallbackText },
});
```

If multiple paths create the same kind of post, extend their existing shared helper where possible. Keep fallback generation alongside the app's post data preparation.

## Verify the result

Use the developer's local project checks and public Reddit pages:

1. Run the app's existing typecheck/build and relevant tests. Where tests already cover post creation, check that the submitted fallback is non-empty and uses the correct public post data. If fallback generation handles answers or private state, check that it excludes them.
2. When live testing is authorized, create a new post through the app on the developer's test subreddit. Open the post on `old.reddit.com` and check that the fallback is readable, Markdown formatting works, and no private content or answers appear.
3. Open the same post on `www.reddit.com` and check that the game still loads and plays. The game may appear instead of the fallback during normal browsing.

Search indexing takes time. A successful build or a readable fallback on Old Reddit does not verify search indexing or screen-reader behavior. Report the creation paths changed, the fallback copy, the checks completed, and any checks that could not be completed. Include the test post URL and app version when live testing was performed.
