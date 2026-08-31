---
name: devvit-journeys
description: 'Design, add, review, or troubleshoot server-authoritative Devvit Journeys telemetry in Devvit Web apps. Use for journey maps, lifecycle events, receipts, allowlisting, or Journey analytics (trigger phrases: "add journeys", "journey map", "journey analytics", "journey receipt", "instrument my game").'
---

# Devvit Journeys

Map a Devvit Web app's actual user flow to Journey events, implement them, and verify that Reddit records them.

**Constraints:**

- Devvit Journeys supports only Devvit Web apps (WebViews). If the app is not a WebView app, explain the limitation and stop.
- Use the official [`reddit/devvit-docs`](https://github.com/reddit/devvit-docs) repository as the source of truth. If `devvit-docs` is installed, use it for version-matched documentation lookup.
- Do not use `Journey.Start` for an app load, component mount, page view, or other passive event.
- Except for `App.Ready`, fire events only after committed user actions or their resulting terminal outcomes. Do not instrument `mouseDown`, `touchStart`, or inputs that can still be canceled.
- Do not put PII or free-form user content in `action`, `actionDetails`, or other user-defined fields. Prefer stable, bounded values.
- Journey maps require review, and events from non-allowlisted apps are dropped. Flag both external requirements when they are unresolved; do not invent an approval or allowlisting process.
- Do not silently upgrade Devvit dependencies. If the installed version does not expose the documented Journeys API, explain the required update before changing versions.

## How It Works

1. Inspect `devvit.json`, `package.json`, the client and server entry points, existing telemetry, and the handlers for the user's actual flow.
2. Read the current official documentation for [Devvit Journeys](https://developers.reddit.com/docs/capabilities/analytics/devvit-journeys), [Journey receipts](https://developers.reddit.com/docs/capabilities/analytics/journeys-receipts), and, when relevant, the [Journeys dashboard](https://developers.reddit.com/docs/capabilities/analytics/journeys-dashboard).
3. Before editing, map concrete code paths to the ready point, explicit start, meaningful progress milestones, useful interactions, and every explicit terminal outcome.
4. If the code leaves multiple plausible journey boundaries that would produce materially different metrics, present the alternatives and ask the user to choose. Otherwise, state the inferred map and continue.
5. Prefer the server-authoritative integration described below. Use client-managed events only where the server cannot observe the event directly, and reuse the app's existing server framework and routing conventions.
6. Instrument only the mapped lifecycle, then run the repository's focused tests, typecheck, and lint or formatting checks.
7. Exercise the real flow when possible and inspect each returned receipt. A successful HTTP response alone does not prove that an event was recorded.

## Map Events to User Behavior

| API call         | Use it when                                                  | Important fields                                                   |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------------ |
| `appReady()`     | The app has finished loading and is interactive              | None                                                               |
| `startJourney()` | The user explicitly begins the mapped experience             | None                                                               |
| `progress()`     | The user reaches a meaningful milestone                      | `progress` from `0.0` to `1.0`; optional `action`, `actionDetails` |
| `interaction()`  | The user completes a useful, stateless interaction           | `action`; optional `actionDetails`                                 |
| `endJourney()`   | The user reaches an explicit success, failure, or quit state | Optional `complete`; optional `game.win` and `game.score`          |

Place calls in the handler that commits the state transition, after the action succeeds. Avoid duplicate instrumentation in both a UI handler and the server operation it invokes.

Call `appReady()` once per app load. Set `complete` explicitly on every end state so completion-rate metrics distinguish a finished experience from failure or an explicit quit.

`progress()` and `interaction()` can auto-start a missing journey, but call `startJourney()` at the intended user start so start-rate metrics retain their meaning.

## Implement the Integration

1. Enable the capability in `devvit.json`:

   ```json
   {
     "permissions": {
       "journeys": true
     }
   }
   ```

2. Confirm that `@devvit/analytics` is available and compatible with the app's Devvit version. Use the project's existing package manager if a dependency change is needed.
3. Prefer the server telemetry API from `@devvit/analytics/server/reddit` for journey starts, milestones, and terminal events. Pass the active client journey ID through the app's existing request, validate the request using existing conventions, and derive the event from trusted server state.
4. Use the client telemetry API from `@devvit/analytics/client/reddit` only for events the server cannot observe directly, such as `appReady()`. Mount the documented telemetry route adapter on the server and let the client manage the active `journeyId`.
5. Do not pass or store the `journeyId` manually for client-managed events, and do not depend on the SDK's storage implementation.
6. Await telemetry calls where their result is used for validation or diagnostics. Do not show receipt messages as end-user product copy.

## Verify Receipts and Analytics

Use `receipt.status` for branching, tests, and tooling; use `receipt.message` only as developer-facing diagnostic text.

- `JOURNEY_RECEIPT_VALID` means the event was accepted and recorded.
- `JOURNEY_RECEIPT_DENIED_NOT_ALLOWLISTED` means the integration reached Reddit, but the app still needs allowlisting.
- `JOURNEY_RECEIPT_DENIED_RATE_LIMITED` and `JOURNEY_RECEIPT_DENIED_DUPLICATE` mean the event was valid but not recorded again.
- `JOURNEY_RECEIPT_INVALID` means the payload was rejected.
- `JOURNEY_RECEIPT_UNSPECIFIED` means Reddit could not confirm whether it recorded the event.

Test at least one complete path from ready through end, plus applicable failure and explicit quit paths. Confirm that repeated play can start a new journey after `endJourney()`.

Use receipts for immediate implementation feedback. The dashboard aggregates data by app and UTC day, so an empty dashboard does not immediately mean the integration failed.

## Present Results to User

- Show the final journey map with the concrete handler or file for each event.
- Summarize configuration, client, server, and test changes.
- Report the receipt statuses you observed, separating implementation success from allowlisting or journey-map approval.
- State which real user paths were exercised and anything that remains unverified.

## Troubleshooting

- **Events are dropped** — Inspect `receipt.status`; the app may be valid but not allowlisted.
- **Start is reused** — The client already has an active journey. Check for a missing terminal `endJourney()` or a duplicate start handler.
- **Progress or interaction starts unexpectedly** — Add the missing explicit `startJourney()` at the intended user action and remove passive instrumentation.
- **Receipt is invalid** — Check required fields, the `0.0` to `1.0` progress range, and non-empty action values.
- **Dashboard is empty** — Verify receipts first, then account for UTC-day aggregation and the app's approval state.
- **App is not a WebView** — Journeys are not supported for that app type.
