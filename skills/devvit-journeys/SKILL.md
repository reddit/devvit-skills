---
name: devvit-journeys
description: 'Design, add, review, or troubleshoot Devvit Journeys in Devvit Web apps. Use for "add journeys", "journey map", "journey analytics", "journey receipts", or "instrument my game".'
---

# Devvit Journeys

Add a thin telemetry wrapper around the app's existing gameplay signals and verify the integration within the app version's approval state. Progress is a core downstream ranking and engagement signal: every integration must include it.

## Constraints

- Journeys supports Devvit Web apps (WebViews) only.
- Use the official [`reddit/devvit-docs`](https://github.com/reddit/devvit-docs) repository and the installed SDK's exports and types. Use `devvit-docs` for version-matched lookup when installed. If documentation and the installed API differ, report the difference rather than copying incompatible calls.
- Do not start journeys on load, mount, page view, or other passive activity. Except for `App.Ready`, emit only after committed user actions or their resulting terminal outcomes, never on cancelable `mouseDown` or `touchStart` input.
- Do not attach PII or free-form user content. Use stable, bounded actions and metadata.
- Recording requires Reddit's approval of the app version and journey map; app allowlisting also applies. Expected approval gating is not an implementation failure. Do not publish, seek approval, or change release state merely to obtain recorded receipts.
- Explain necessary SDK or peer-dependency updates before changing versions. Preserve the app's package-manager convention and update its maintained lockfiles; do not silently leave a second maintained lockfile inconsistent.

## How It Works

1. Inspect `devvit.json`, dependencies and lockfiles, client/server entrypoints, existing telemetry, and actual start, progress, result, reset, and navigation handlers. Capture baseline test/typecheck/lint failures so existing defects are not attributed to the integration.
2. Read [Devvit Journeys](https://developers.reddit.com/docs/capabilities/analytics/devvit-journeys) and [Journey receipts](https://developers.reddit.com/docs/capabilities/analytics/journeys-receipts). Read the [dashboard documentation](https://developers.reddit.com/docs/capabilities/analytics/journeys-dashboard) when checking aggregate data.
3. Before editing, map each event to an existing handler or state transition, citing its file/function and the signal it exposes. Reuse existing game helpers, constants, and SDK facilities. Mark missing or ambiguous signals as blocked under the scope rules below; do not fill the gaps with new gameplay logic. Account for tutorial, guest, retry, and resumed-play paths that the app actually supports.
4. Check SDK compatibility, choose event ownership, and integrate using the guidance below. Read [framework integration](references/framework-integration.md) when mounting client routes or mixing server and client events.
5. Test the mapped lifecycle, including ordering and failure behavior, then run the relevant repository checks and build.
6. Exercise the real flow when available. Save receipt diagnostics and distinguish local tests, live integration, and recorded analytics. Stop at an expected approval gate after verifying the permitted paths.

## Scope: Instrument Existing Behavior

Implement telemetry calls, event metadata, SDK/configuration wiring, necessary adapters, and telemetry-specific ordering, deduplication, diagnostics, and failure isolation. Keep these changes small and reuse existing facilities. Do not implement or duplicate game rules, win/completion detection, scoring, reset/replay policies, persistence models, UI controls, or a parallel gameplay state machine to make a journey fit. Optional result fields must come from existing game results; omit them when absent. A telemetry guard may prevent duplicate emission from an existing transition; it must not invent that transition. If attaching events would require restructuring gameplay or state management, report that prerequisite instead of performing the rewrite.

When a required signal is missing or ambiguous, tell the developer which event could not be placed, what code you inspected, and what decision or app-owned signal is needed. Defer the affected journey lifecycle until the developer supplies a clear existing hook or separately requests the prerequisite gameplay work. Do not silently ship Start/Progress as a complete integration when End cannot be mapped. Independent, clearly supported instrumentation such as Ready may be added, but label the result partial and avoid unused routes or scaffolding. A missing optional interaction or score need not block an otherwise complete map.

Offer at most a brief, context-specific suggestion, without implementing it: “Could not add Journey.End: this Bingo app saves marked tiles but has no completion signal. If a completed line should end play, the game needs an explicit completion callback that telemetry can observe.” A product convention or a plausible metric is not evidence that the app already implements that rule. Required Progress does not override these scope limits: a fixed `0.5` engagement milestone may observe an existing committed action, but cannot justify inventing an End or new gameplay behavior.

## Map Events to User Behavior

| Event            | Concrete trigger                                                        | Payload                                                            |
| ---------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `appReady()`     | The relevant entrypoint has usable data and is interactive              | None                                                               |
| `startJourney()` | The user commits to a new attempt                                       | None                                                               |
| `progress()`     | A committed milestone; at least one per started journey                 | `progress` in `[0, 1]`; optional bounded `action`, `actionDetails` |
| `interaction()`  | A useful committed action within the mapped experience                  | Bounded `action`; optional `actionDetails`                         |
| `endJourney()`   | Success, failure, explicit quit, or confirmed replacement of an attempt | Explicit `complete`; optional `game.win`, `game.score`             |

Instrument the handler that commits the transition, not both that handler and a rendering effect. `complete` expresses completion of the mapped experience; do not equate every terminal state with success. Set it explicitly for every end path. A restart should end an unfinished attempt before starting another; viewing saved results must not create a new completion.

Always map at least one `progress()` event into each started journey, ordered after Start and before End. Prefer a natural completion fraction when one exists. Otherwise, use a fixed milestone such as `progress: 0.5` on the first committed gameplay action, with a bounded action such as `play_engaged`. Document that this is an engagement milestone, not a claim that the player is literally halfway finished. It can remain at `0.5` until End; there is no requirement to manufacture further percentages or send `1.0`.

The same committed action may explicitly start the journey and then emit its first progress event. This covers short attempts and immediate wins without starting on load. Emit the fallback milestone once per journey, reset that guard for replay, and keep failed-start handling intact: do not let progress auto-start another journey. Interactions may supplement progress but must not replace it. For a game that already exposes explicit start and completion hooks, the first committed play action can send Start, Progress (`0.5`, `play_engaged`), then its interaction; the existing completion hook sends End.

Call ready once per relevant app load, including framework remount behavior. Do not fire ready while loading or on a failed/empty game state. For multiple entrypoints, state which load is measured so readiness counts are not accidentally doubled.

Always start explicitly. Auto-start behavior varies by SDK: in 0.14.3, `progress()` may start a journey while `interaction()` can send without one. Do not rely on either method to establish the intended boundary.

## SDK and Event Ownership

Enable `permissions.journeys: true` in `devvit.json` and declare `@devvit/analytics` directly if the app imports it.

Check the installed declarations before coding. SDK 0.14.3 was verified to expose `startJourney(): Promise<{ journeyId, receipt }>` and `{ receipt }` from the other calls. Older builds can return a string from start and void from other methods. 0.14.3 is a tested reference, not a claimed minimum or a reason to downgrade a newer app. Align Devvit packages and inspect peer requirements when an update is needed; `@devvit/start@0.14.3` requires Vite >=7.3.5.

Choose ownership from what the app can actually observe:

- **Server-observed transitions:** Prefer `@devvit/analytics/server/reddit` after validated gameplay commits. Use the game's existing results from trusted state and reuse existing request validation and routing. Receiving a client-supplied score does not make it authoritative.
- **Client-only gameplay:** Use `@devvit/analytics/client/reddit` with matching server routes. A local game or guest session need not acquire a server gameplay model just to add telemetry. Delayed result uploads after login are persistence, not new play or new completions.
- **Mixed ownership:** Carry the SDK's active ID through the existing request and define who starts, ends, and clears it. Never emit the same transition from both sides. See the reference for ID handoff and late-response handling.

For client-managed events, let the SDK own the ID. Do not implement separate ID storage or rely on the SDK's storage key. Follow the game's existing reload/attempt boundaries; report ambiguity rather than choosing new ones: resume only if gameplay resumes; a new explicit attempt must not reuse an abandoned stored journey. Use public SDK session methods when reconciling that state.

## Ordering and Failure Handling

Telemetry must not control gameplay success or block saving a result. Await calls for ordering and diagnostics inside a telemetry task, with failures contained; do not make the user wait for analytics.

Keep start, interactions, end, and replay ordered when requests overlap. Guard repeated terminal callbacks and committed start actions without suppressing an intentional replay. Keep side effects out of React state-updater functions, which may be invoked more than once.

Bound transport waits and prevent unbounded queued work during an outage. A bare `Promise.race` does not cancel the losing request: a late start may still overwrite the SDK's active ID. Prefer aborting the underlying fetch or another SDK-supported cancellation mechanism. After a failed start, do not let a later progress call auto-start a different attempt. Verify that a failed end cannot leave the next attempt attached to the previous ID.

## Verification and Receipts

Use `receipt.status` for application/test branching and `receipt.message` for developer diagnostics. Do not use exact message text as a programmatic API contract or show receipt text to players. Log a serialized record containing the event, receipt status/message, and returned journey ID when available (for example, `JSON.stringify({ event, ...response })`). Passing only an object to `console.info` can collapse to `Object` in exported browser/CLI logs, hiding the evidence needed for verification.

| Status                                                                     | Interpretation                                                                |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `JOURNEY_RECEIPT_VALID`                                                    | Accepted and recorded                                                         |
| `JOURNEY_RECEIPT_DENIED_NOT_ALLOWLISTED`                                   | Reached Reddit; app is not enabled for recording                              |
| `JOURNEY_RECEIPT_DENIED_RATE_LIMITED` / `JOURNEY_RECEIPT_DENIED_DUPLICATE` | Skipped; do not retry blindly or count as another recorded event              |
| `JOURNEY_RECEIPT_INVALID`                                                  | Payload rejected; investigate the integration                                 |
| `JOURNEY_RECEIPT_UNSPECIFIED`                                              | Recording not confirmed; inspect diagnostics and the version's approval state |

A playtest version can return UNSPECIFIED with the diagnostic `Denied: Event was valid, but Journey telemetry is not recorded for playtest app versions.` That is expected approval gating, not evidence of a code defect or missing app allowlisting. Capture the status and diagnostic, report that live integration was exercised, and leave ingestion verification pending an approved version. Do not treat every UNSPECIFIED response as this case: transport failures and other unknown outcomes still need investigation.

Verify at least one Progress event between Start and End on success, applicable failure/quit, and replay, including a one-action completion. Check the fallback milestone fires once per journey, resets for replay, and does not emit on passive load or after a failed start. Test the boundaries most likely to duplicate or misattribute events: remount, canceled confirmation, rapid start/end/restart, stale session, and failed requests. Verify both the actual UI/handler wiring and the client/server contract. Mock receipts prove only local behavior; label them as simulated. Compare repository checks with the baseline.

Use three separate outcomes in the result:

1. **Local implementation:** which paths and tests passed, and any new versus existing check failures.
2. **Live integration:** the paths actually exercised and the returned statuses/diagnostics, including expected approval gating.
3. **Recorded analytics:** observed VALID receipts from an eligible version, or explicitly pending verification. The dashboard aggregates by app and UTC day, so an empty dashboard alone does not diagnose integration failure.

## Present Results to User

First report whether the lifecycle was fully instrumented or blocked by missing app signals. For a blocked event, state “could not add,” explain the missing hook, and give a short developer suggestion rather than claiming success. Show the journey map with concrete handlers/files, summarize configuration and dependency changes, link logs and test evidence, and identify any real paths left untested. Treat expected approval gates as deployment prerequisites, not a reason to undo a working integration or inflate claims of recording.
