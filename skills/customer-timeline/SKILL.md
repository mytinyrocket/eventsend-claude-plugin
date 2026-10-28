---
name: customer-timeline
description: Reconstruct what one customer did from their EventSend events. Use when the user asks what a customer, user or account did, what happened before they churned, cancelled, upgraded or had a payment fail, or gives a customer id, Stripe customer id (cus_…) or email to look up.
---

# Reconstruct a customer's timeline

EventSend ties events to a customer through `user_id`: an identifier the sender chooses and sends
with every event about that customer. It is matched exactly and case-sensitively, so `cus_9Xk2` and
`CUS_9XK2` are two different customers. Stripe sources always set it to the Stripe customer id
(`cus_…`), never to an email.

## 1. Find the identifiers

- **The user gave an id**: use it as given.
- **The user gave an email or a name**: emails are not `user_id`s and payloads are not searched.
  Call `search_events` with the email or name as `search`, which matches event names and messages,
  plus `include_payload: true`. Take the `user_id` from any event that matches. If nothing matches,
  ask the user for the customer id and explain why.
- **Several systems report on the same person** (a Stripe `cus_…` and your own account id, for
  example): pass all of them together, up to five.

If the user names an organization, call `list_organizations` first and pass its slug as `org`.

## 2. Read the timeline

Call `get_user_timeline` with `user_ids`, and `since` or `until` if the user gave a period. The
first page carries a `summary`: the total, first and last seen, and counts by level and by event
name. Use it to size the story before reading individual events.

- Use `limit: 200` when the user wants the whole history. Keep paging with `next_cursor` only while
  `exhausted` is false and the answer still needs older events.
- Timeline rows leave out payloads. Call `get_event` for the few events whose details matter, such
  as a failed payment's amount or a cancellation reason. Don't fetch every event.
- Events are ordered by when they happened (`occurred_at`), so a late webhook lands in the right
  place.

## 3. Tell the story

Present the events oldest to newest, even though the tool returns them newest first.

- Open with one or two sentences: who this is, the span covered, and the outcome the user asked
  about ("signed up 14 Aug, paid twice, card failed 2 Sep, cancelled 4 Sep").
- Group bursts of similar events ("12 `api.request` events on 3 Sep") rather than listing each one.
- Call out money moving, errors and `critical` events, plan changes, and long silences.
- Give dates and times in the organization's time zone, which every response includes.
- Raw events are kept for the plan's retention window (`retention_days`). If the story starts at
  the edge of that window, say that earlier history may exist but is no longer visible.

Payloads can hold personal data. Quote only the fields that answer the question.
