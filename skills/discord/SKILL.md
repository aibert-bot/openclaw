---
name: discord
description: "Discord ops via the message tool (channel=discord)."
metadata: { "openclaw": { "emoji": "🎮", "requires": { "config": ["channels.discord.token"] } } }
allowed-tools: ["message"]
---

# Discord (Via `message`)

Use the `message` tool. No provider-specific `discord` tool exposed to the agent.

## Musts

- Always: `channel: "discord"`.
- Respect gating: `channels.discord.actions.*` (some default off: `roles`, `moderation`, `presence`, `channels`).
- Prefer explicit ids: `guildId`, `channelId`, `messageId`, `userId`.
- Multi-account: optional `accountId`.

## Guidelines

- Avoid Markdown tables in outbound Discord messages.
- Mention users as `<@USER_ID>`.
- Prefer Discord components v2 (`components`) for rich UI; use legacy `embeds` only when you must.

## Targets

- Send-like actions: `to: "channel:<id>"` or `to: "user:<id>"`.
- Message-specific actions: `channelId: "<id>"` (or `to`) + `messageId: "<id>"`.

## Common Actions (Examples)

Send message:

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "hello",
  "silent": true
}
```

Send with media:

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "see attachment",
  "media": "file:///tmp/example.png"
}
```

- Optional `silent: true` to suppress Discord notifications.

Send with components v2 (recommended for rich UI):

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "Status update",
  "components": "[Carbon v2 components]"
}
```

- `components` expects Carbon component instances (Container, TextDisplay, etc.) from JS/TS integrations.
- Do not combine `components` with `embeds` (Discord rejects v2 + embeds).

Legacy embeds (not recommended):

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "Status update",
  "embeds": [{ "title": "Legacy", "description": "Embeds are legacy." }]
}
```

- `embeds` are ignored when components v2 are present.

React:

```json
{
  "action": "react",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456",
  "emoji": "✅"
}
```

Read:

```json
{
  "action": "read",
  "channel": "discord",
  "to": "channel:123",
  "limit": 20
}
```

Edit / delete:

```json
{
  "action": "edit",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456",
  "message": "fixed typo"
}
```

```json
{
  "action": "delete",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456"
}
```

Poll:

```json
{
  "action": "poll",
  "channel": "discord",
  "to": "channel:123",
  "pollQuestion": "Lunch?",
  "pollOption": ["Pizza", "Sushi", "Salad"],
  "pollMulti": false,
  "pollDurationHours": 24
}
```

Pins:

```json
{
  "action": "pin",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456"
}
```

Threads:

```json
{
  "action": "thread-create",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456",
  "threadName": "bug triage"
}
```

Search:

```json
{
  "action": "search",
  "channel": "discord",
  "guildId": "999",
  "query": "release notes",
  "channelIds": ["123", "456"],
  "limit": 10
}
```

Presence (often gated):

```json
{
  "action": "set-presence",
  "channel": "discord",
  "activityType": "playing",
  "activityName": "with fire",
  "status": "online"
}
```

## Writing Style (Discord)

- Short, conversational, low ceremony.
- No markdown tables.
- Mention users as `<@USER_ID>`.

---

## Common Workflows

### Create a Channel in a Campaign Category

1. **Find the category ID** — get it from an existing channel in the category:
```json
{"action": "channel-info", "channel": "discord", "target": "<existing_channel_id>"}
```
The `parent_id` in the response is the category ID.

2. **Create the channel:**
```json
{"action": "channel-create", "channel": "discord", "guildId": "1467597829097914627", "name": "my-channel", "topic": "Channel purpose"}
```

3. **Move it into the category** (channel-create doesn't support parentId directly):
```json
{"action": "channel-edit", "channel": "discord", "channelId": "<new_channel_id>", "parentId": "<category_id>"}
```

4. **Post an initial message:**
```json
{"action": "send", "channel": "discord", "target": "channel:<new_channel_id>", "message": "Channel is live. Purpose: ..."}
```

### Scan a Category for Updates

Read recent messages from all channels in a campaign to understand current state:

1. **List channels** to find all in the category (or use known IDs from CAMPAIGN.md)
2. **Read each channel:**
```json
{"action": "read", "channel": "discord", "target": "channel:<id>", "limit": 10}
```
3. Summarize what's happening across channels — look for blockers, decisions, progress.

### Post a Status Update to Multiple Channels

When a DAG node completes or a gate activates, post to both #control and #dag:

```json
{"action": "send", "channel": "discord", "target": "channel:<control_id>", "message": "..."}
```
```json
{"action": "send", "channel": "discord", "target": "channel:<dag_id>", "message": "..."}
```

### Create a Category

```json
{"action": "category-create", "channel": "discord", "guildId": "1467597829097914627", "name": "my-campaign"}
```
Returns the category with its ID. Use this ID as `parentId` when creating/moving channels.

### React to Acknowledge

When a user posts something that doesn't need a text reply, react instead:
```json
{"action": "react", "channel": "discord", "channelId": "<channel_id>", "messageId": "<msg_id>", "emoji": "✅"}
```

### Search for Prior Decisions

Find old messages about a topic across channels:
```json
{"action": "search", "channel": "discord", "guildId": "1467597829097914627", "query": "decided to use", "channelIds": ["<id1>", "<id2>"], "limit": 10}
```

## Key IDs (Aibert-Prime)

- **Guild:** `1467597829097914627`
- **#control:** `1469424689699881053`
- **Albert:** `172966866195709952`
- **Travel category:** `1472363807467638845`
- **Travel #control:** `1472363840825200845`
- **Travel #dag:** `1476357206160183576`
