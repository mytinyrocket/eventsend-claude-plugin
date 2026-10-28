---
name: investigate-failures
description: Find out why EventSend notifications failed or never arrived. Use when the user asks why an alert didn't show up in Slack, Discord, Telegram, a webhook or push, whether deliveries are failing, what is wrong with a destination, or what errors happened recently.
---

# Investigate failed or missing notifications

Every tool on the EventSend connector is read-only. You can diagnose, but you cannot retry, replay,
re-enable or edit anything. End with what the user should change in EventSend and where.

## 1. Pin down the question

- **Organization**: if the user names one, call `list_organizations` and pass its slug as `org` on
  every later call.
- **Destination**: if the user names a channel or endpoint ("the #billing Slack", "our webhook"),
  call `describe_setup` and take the destination's `dst_…` id from it. Never guess an id.
- **Window**: default to the last 7 days. "Today" is `days: 1`; a specific day is `from` and `to`
  set to that date.

## 2. Check delivery health first

Call `delivery_health` with the window (and `destination_id` when there is one). For each
destination, read `delivered`, `failed`, `success_rate` and `skipped`, then `recent_failures`.

- A destination with `active: false` was switched off; a webhook that answered 401, 403 or 404 is
  deactivated automatically.
- `skipped.quiet_hours` and `skipped.rollup` are deliberate holds (quiet hours, or events folded
  into a rollup message), not failures.
- A deleted destination still appears in the counts as `deleted: true`.

## 3. Follow one missing event

When the user means one notification ("the payment alert from this morning never came"), call
`search_events` with the event name or a word from its message and a `since` window, then read
the event's `status`:

| `status` | Meaning |
| --- | --- |
| `delivered` | Every delivery succeeded. Check that the user is looking at the right channel. |
| `partial` | Some deliveries succeeded and others failed or have an unknown outcome. |
| `failed` | No delivery succeeded. Read the errors with `get_event`. This includes deliveries skipped as too old to send: nothing goes out more than 24 hours after the event, and Slack's limit is shorter (15 minutes for `critical`, an hour for `error` and `warning`, six hours otherwise). |
| `delivery_unknown` | Slack only: the message may or may not have posted, so it was not retried automatically. |
| `dropped` | No active topic matched, so nothing was sent. Explain the routing with `describe_setup`: a topic's `min_level` or `event_pattern` excluded it, or no topic connects the route to that destination. |
| `skipped` | Held on purpose everywhere: quiet hours, or folded into a rollup message. Not a failure. |
| `pending` / `processing` | Still in flight or waiting for a retry. Retries run for about 15 minutes. |

If no event turns up at all, it probably never reached EventSend. Check `describe_setup` for a
route with `active: false` or `debug_mode: true` (debug routes record nothing), and `get_usage`
for a plan that has used up its monthly allowance.

## 4. Read the errors

Call `get_event` on at most three representative failing events, unless the user asks for more.
Each delivery carries `status_code`, `error_message`, `attempts`, `skip_reason` and its
destination. Group failures by destination and by error, and quote error messages exactly.

Common causes:

- **Webhook 5xx, timeout, connection refused**: the endpoint was down. Retries are spent after
  about 15 minutes.
- **Webhook 401/403/404**: the URL or its auth changed, and the destination was deactivated.
- **Webhook 400/422**: the endpoint rejects the payload. EventSend does not retry these.
- **Slack, Discord or Telegram credential errors**: the app was removed from the workspace or chat,
  or the channel was deleted or archived. Reconnect under Connections.
- **429**: the receiver is rate limiting.

## 5. Report

Lead with a one-line verdict: healthy, degraded or failing. Then:

1. A short table of the affected destinations: delivered, failed, success rate, last failure.
2. The top errors with counts, the exact message, and when they first and last appeared.
3. The likely cause and the fix, naming the page in EventSend where the user can make it:
   Destinations to re-enable or edit, Connections to reconnect, Topics for routing rules. Failed
   deliveries can be resent from the event's page, or in bulk from the destination's Replays page,
   once the cause is fixed.

Give times in the organization's time zone, which every response includes.
