---
name: activity-report
description: Summarize what happened in an EventSend organization over a period. Use when the user asks for a daily or weekly summary, what happened today or yesterday, how this week compares with last week, which events are the most common or noisiest, or whether they will stay within their event quota.
---

# Summarize an organization's activity

If the user names an organization, call `list_organizations` first and pass its slug as `org` on
every call.

## 1. Choose the period

Analytics tools take calendar dates in the organization's time zone, not rolling hours:

- "Today": `days: 1`. The response carries `partial_period: true` because today is still filling,
  so say "so far today".
- "Yesterday": `from` and `to` both set to yesterday's date.
- "This week" or "the last week": `days: 7`. "This month": `from` set to the first of the month.

Daily counts are kept for 12 months on every plan. They count events by when EventSend recorded
them (`time_basis: "recorded"`), which can differ slightly from when they happened.

## 2. Gather the numbers

Make these calls with the same period:

1. `event_stats`: total, `by_day`, `by_level` and `previous_period` with its `pct_change`.
2. `list_event_names` with `limit: 10`: the main event names and their counts.
3. `delivery_health`: delivered, failed and success rate per destination, plus recent failures.
4. `get_usage`: the quota used in the current billing period, and the projection (`on_track`).

Then go further only where something stands out:

- An event name that moved sharply: `event_stats` with that `event_name`, at most three of these.
- Any `error` or `critical` events: `search_events` with that `level` and a matching `since`
  (`24h` for today, `7d` for a week), so you can name what went wrong instead of only counting it.

## 3. Write the report

Keep it short enough to read in a minute:

1. **Headline**: one sentence on the period ("A normal week: 4,210 events, up 8% on last week, one
   delivery problem").
2. **Volume**: the total against the previous period, and the busiest day.
3. **What happened**: the top event names in plain language, and any that rose or fell notably.
4. **Problems**: errors and critical events by name, and any destination below 100% delivery with
   its main error. If there are none, say so in one line.
5. **Quota**: the percentage used, and whether the projection stays within the limit before the
   period ends.

Compare against the previous period only when `previous_period.available` is true. Round
percentages, give dates in the organization's time zone, and mark partial days as partial.
