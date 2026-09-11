# Framework integration

Inspect the installed SDK before choosing an adapter. The examples below use public APIs verified in `@devvit/analytics@0.14.3`; newer versions may provide additional adapters.

## Express

`createTelemetryRouter()` returns an Express router with JSON parsing and the default `/api/telemetry` prefix already included:

```ts
import { createTelemetryRouter } from "@devvit/analytics/server/reddit";

app.use(createTelemetryRouter());
```

Do not mount it at `/api/telemetry` again unless you also change its `basePath` to avoid a doubled prefix. Match a custom client `basePath` to the actual mounted paths.

## Hono and other frameworks

SDK 0.14.3's Express router is not Hono middleware. Its `server/reddit` entrypoint also re-exports that router, so bundling can still require the declared `express` peer even when the app uses Hono. Check the package's peer dependencies and satisfy them with the project's package manager; npm may install peers automatically while Yarn Classic does not. Preserve the app's framework. If the installed SDK has no matching adapter, implement small routes using its public server API and the shared endpoint constants. Keep the existing middleware/context and validation conventions; do not replace the server framework.

The full client contract at the default `/api/telemetry` base path is:

| Shared constant in `TELEMETRY_JOURNEY_ENDPOINTS` | Relative POST path     | Server method                         |
| ------------------------------------------------ | ---------------------- | ------------------------------------- |
| `appReady`                                       | `/journey/app-ready`   | `telemetry.appReady()`                |
| `start`                                          | `/journey/start`       | `telemetry.startJourney()`            |
| `progress`                                       | `/journey/progress`    | `telemetry.journeyProgress(input)`    |
| `interaction`                                    | `/journey/interaction` | `telemetry.journeyInteraction(input)` |
| `end`                                            | `/journey/end`         | `telemetry.endJourney(input)`         |

Implement only routes the mapped integration uses. In the server API, progress and interaction method names differ from those on the client. Return the complete SDK response so the client receives both the start ID and receipts. Do not reduce it to `{ ok: true }`.

This Hono example mounts the two bodyless calls:

```ts
import { Hono } from "hono";
import { telemetry } from "@devvit/analytics/server/reddit";
import { TELEMETRY_JOURNEY_ENDPOINTS as endpoints } from "@devvit/analytics/shared/reddit";

const journeys = new Hono();
journeys.post(endpoints.appReady, async (c) =>
  c.json(await telemetry.appReady()),
);
journeys.post(endpoints.start, async (c) =>
  c.json(await telemetry.startJourney()),
);

// Add validated progress, interaction, and end handlers used by the map.
app.route("/api/telemetry", journeys);
```

For body-bearing handlers, use the app's schema library. Reject malformed JSON, absent/invalid IDs where required, non-finite or out-of-range progress, invalid completion/game fields, and unexpected fields. Restrict actions and details to the app's bounded vocabulary rather than forwarding arbitrary user strings. The server interaction API expects `actionDetails` and may need `''` when omitted. Use installed types to construct the exact request.

Preserve the receipt envelope for adapter-generated errors too. For invalid input, return HTTP 400 with an error and `{ receipt: { status: 'JOURNEY_RECEIPT_INVALID', message: 'Invalid journey payload.' } }`. Handle service exceptions as a failed request or UNSPECIFIED, never as VALID. Distinguish those failures from a successful SDK response whose receipt reports an expected approval gate.

Contract-test the actual SDK client against these routes with a mocked platform boundary. Check the mounted prefix, payload validation, receipt propagation, and SDK ID clearing/replay. Separately exercise the UI handlers; route-only tests do not prove event placement. The browser SDK uses conditional exports: ensure client tests resolve its browser entrypoint (or mock it at the intended boundary), including existing tests that import the app. Keep server tests on server conditions.

## Mixed server/client ownership

Use this only when the server observes a real gameplay transition. Do not add a second end event to an existing client-managed end.

- If the client starts, await that start in the telemetry sequence before reading `getActiveJourneyId()` and correlating a server event. Do not delay or suppress the gameplay operation if telemetry fails.
- If the server starts, return `{ journeyId, receipt }` and install that ID with `setJourneyId()` only if the response still belongs to the same active gameplay attempt.
- If the server ends, return its receipt and clear the SDK ID with `clearJourneyId()` when that local attempt concludes. Check that the active ID still matches the ended attempt before clearing it; a late response must not erase a newer attempt.
- Keep the SDK as the only ID storage owner. Explicit handoff through its public methods is different from duplicating its storage implementation.
- Make duplicate gameplay submissions safe using the app's existing attempt/idempotency mechanisms. An analytics denial must not roll back a successfully saved game.
