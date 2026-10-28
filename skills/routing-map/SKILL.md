---
name: routing-map
description: Explain how an EventSend organization is set up and where its events go. Use when the user asks which routes, sources, topics or destinations exist, where a given event is sent, why an event did or did not reach a channel, or wants an overview of their EventSend configuration.
---

# Map an organization's routing

If the user names an organization, call `list_organizations` first and pass its slug as `org`.

## How EventSend routes an event

- A **route** is an ingest credential. Events sent from the user's own code arrive on a route.
- A **source** is a provider connector (Stripe, GitHub or Vercel) that feeds events into a route.
- A **topic** connects routes to destinations and holds the routing rules. An event goes through
  every topic its route is connected to.
- A **destination** is where messages are delivered: Slack, Discord, Telegram, a webhook, Pushover
  or browser push.

A topic delivers an event only when both of its rules match:

- `min_level`: levels run `default` → `success` → `warning` → `error` → `critical`, and the topic
  accepts its minimum level and everything more severe.
- `event_pattern`: matches the whole event name, ignoring case. `*` matches any run of characters,
  and a blank pattern matches every name. `payment-*` matches `payment-failed` but not
  `user-created`.

An event that no active topic matches is still recorded. It is marked `dropped` and nothing is sent.

## 1. Read the setup

Call `describe_setup`. It returns routes, sources, topics and destinations with their ids and names,
and never any tokens, secrets or destination settings. If any entry in `truncated` is true, say
that list is incomplete.

## 2. Answer the question

- **"What's set up?"**: draw the map from routes through topics to destinations, as a short nested
  list or a table. Mention anything inactive (`active: false`), routes in `debug_mode` (they accept
  events but record nothing), and destinations whose `health_status` is `needs_attention`,
  `paused` or `pending_test`.
- **"Where does `payment-failed` go?"**: find the route it arrives on (from `search_events`, or the
  source that emits it), then check each of that route's topics against the event's name and level.
  List the destinations of every topic that matches, and name the rule that excludes each topic that
  doesn't.
- **"Why didn't it reach #alerts?"**: do the same check for that one destination, then confirm what
  actually happened with the `investigate-failures` skill's steps.

## 3. Changes

These tools are read-only. To change routing, point the user to the matching page in EventSend:
Topics for rules and attachments, Destinations for channels and endpoints, Routes for ingest
tokens and debug mode, and Sources for provider connectors.
