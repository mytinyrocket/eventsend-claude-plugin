# EventSend for Claude

[EventSend](https://eventsend.io) routes your product's events, like payments, signups, deploys and
errors, to Slack, Discord, Telegram, webhooks and push notifications. This plugin lets Claude answer
questions about those events in plain language: what happened, to whom, and whether the
notifications got through.

It connects Claude to EventSend's read-only MCP server and adds four skills that teach Claude how to
investigate with it. Nothing it does can change your EventSend account.

## What you can ask

- "Did anything fail in checkout today?"
- "Why didn't the payment alert reach #billing this morning?"
- "What did customer `cus_9Xk2` do before they cancelled?"
- "Give me a summary of last week compared with the week before."
- "Where do `deploy.failed` events go?"
- "Are we going to run out of events before the month ends?"

## Skills

| Skill | What it does |
| --- | --- |
| `investigate-failures` | Finds out why a notification failed or never arrived: delivery health per destination, the event's status, the exact errors, and what to fix in EventSend. |
| `customer-timeline` | Rebuilds one customer's history from every event that carries their `user_id`, oldest first, with the moments that matter called out. |
| `activity-report` | Summarizes a day, a week or a month: volume against the previous period, top events, errors, delivery problems and quota. |
| `routing-map` | Explains your routes, sources, topics and destinations, and traces where a given event is delivered and why. |

Claude picks the right skill from your question. In Claude Code you can also run one directly, for
example `/eventsend:activity-report`.

## Install

You need an EventSend account. Every plan works, including Free.

**claude.ai and Cowork**: add EventSend from the plugin directory in **Customize → Plugins**. Then
open the plugin's **Connectors** tab, connect EventSend, sign in, and choose which of your
organizations Claude may read.

**Claude Code**:

```bash
claude plugin marketplace add mytinyrocket/eventsend-claude-plugin
claude plugin install eventsend@eventsend
```

Then run `/mcp`, select `eventsend` and sign in. Your browser opens EventSend, where you choose the
organizations to connect.

**Gemini CLI**: this repository is also a Gemini CLI extension.

```bash
gemini extensions install https://github.com/mytinyrocket/eventsend-claude-plugin
```

Then run `/mcp auth eventsend` and sign in.

**Cursor, ChatGPT and other clients**: the same server works in any client that supports remote MCP
servers. See the [client setup guide](https://eventsend.io/docs/mcp) for each one.

## Data and privacy

The plugin runs no code of its own and stores nothing. It contains instructions for Claude and one
connector reference, to `https://eventsend.io/mcp`.

When Claude uses an EventSend tool, the tool's arguments go to EventSend: search text, event names,
customer identifiers, organization names and dates. What comes back is your organization's data,
which then becomes part of your conversation with Claude. That covers events and their payloads,
delivery records and errors, daily counts, quota, and the names and ids of your routes, sources,
topics and destinations. No tool returns ingest tokens, signing secrets, MCP keys or destination
settings, and no tool can create, change or delete anything.

Sign-in uses OAuth. A connection sees exactly what you can see in the organizations you approved. To
revoke it, disconnect under **Connected apps** in your EventSend settings, or ask an organization
owner or admin to remove it under **Settings → MCP**. Revocation takes effect on the next request.

- [Privacy policy](https://eventsend.io/privacy)
- [Terms of service](https://eventsend.io/terms)
- [MCP server documentation](https://eventsend.io/docs/mcp)

## Support

Open an issue in this repository, or email [support@eventsend.io](mailto:support@eventsend.io).

## License

[MIT](LICENSE)
