# workflow-pilot

MCP-based workflow orchestrator. Small, boring, and built to run unattended.

v1 target: aggregate the morning of a solo developer into one job — email, calendar
and GitHub activity in, structured notes out to Notion and a chat message.

## Layout

Monorepo, two packages:

| Package | Responsibility |
|---|---|
| `pilot-core` | Scheduler state machine. Triggers: cron, signed webhook, conversation. |
| `pilot-connectors` | One adapter per service: GitHub issues→PR, Gmail unread, Notion write, Lark/Slack notify. |

## Design notes

- **Triggers** — a run starts from cron, an inbound webhook, or an explicit chat
  request. All three feed the same state machine so retry and resume behaviour is
  identical.
- **Webhook auth** — HMAC-SHA256 over the raw body, signature compared in constant
  time, timestamp window checked to reject replays.
- **Per-connector tokens** — each connector holds its own credential with the
  narrowest scope that works. Rotation is per-connector, so a leak costs one
  integration, not the whole stack.
- **Rate limits** — requests are paced against each provider's published quota;
  the scheduler defers rather than retries into a 429.
- **Idempotency** — events are keyed by `event_id` and deduped in a local store, so
  a redelivered webhook does not run the job twice.
- **Sandbox** — connectors declare a scope allowlist; anything outside it is
  refused before the request is made.
- **CI** — GitHub Actions runs lint and vitest on every PR, plus a contract test
  against recorded mocks.

## Status

v1 in progress. The first scenario is the morning digest: 08:00 local, pull
unread mail and today's calendar and open GitHub items, summarise into one Notion
page, post a short version to the team chat.

## License

MIT
