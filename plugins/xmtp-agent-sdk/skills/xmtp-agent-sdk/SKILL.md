---
name: xmtp-agent-sdk
description: Use when building, extending, or modifying XMTP messaging agents using @xmtp/agent-sdk, or when working with XMTP event handlers, middleware, message content types, or agent lifecycle
---

# XMTP Agent SDK

Event-driven, middleware-powered messaging agents on XMTP. Install: `npm install @xmtp/agent-sdk`

## Quick Start

```ts
import { Agent, createUser, createSigner, getTestUrl } from "@xmtp/agent-sdk";

const agent = await Agent.create(createSigner(createUser()), {
  env: "dev",
  dbPath: null,
});

agent.on("text", async (ctx) => {
  await ctx.sendTextReply(`Echo: ${ctx.message.content}`);
});

agent.on("start", (ctx) => console.log(getTestUrl(ctx.client)));
await agent.start();
```

Or use env vars (`XMTP_WALLET_KEY`, `XMTP_ENV`, `XMTP_DB_DIRECTORY`, `XMTP_DB_ENCRYPTION_KEY`):

```ts
process.loadEnvFile(".env");
const agent = await Agent.createFromEnv();
```

## Events

Register with `agent.on(event, handler)`. Self-messages and undefined content are auto-filtered.

**Message:** `text`, `markdown`, `reaction`, `reply`, `attachment`, `inline-attachment`, `multi-attachment`, `actions`, `intent`, `read-receipt`, `transaction-reference`, `wallet-send-calls`, `group-update`, `leave-request`, `message` (all), `unknownMessage`

**Conversation:** `conversation`, `dm`, `group`

**Lifecycle:** `start`, `stop`, `unhandledError`

**Warning:** `"message"` fires for every type. Always filter to prevent infinite loops:

```ts
import { filter } from "@xmtp/agent-sdk";
agent.on("message", async (ctx) => {
  if (filter.isText(ctx.message)) {
    await ctx.conversation.send(`Echo: ${ctx.message.content}`);
  }
});
```

## Middleware

`(ctx, next)` — call `next()` to continue, `return` to stop, `throw` for error middleware.

```ts
const onlyText: AgentMiddleware = async (ctx, next) => {
  if (filter.isText(ctx.message)) await next();
};
agent.use(onlyText);
```

**CommandRouter** for slash commands:

```ts
const router = new CommandRouter({ helpCommand: "/help" });
router.command("/ping", "Pong!", async (ctx) => {
  await ctx.conversation.sendText("Pong!");
});
agent.use(router.middleware());
```

**Error middleware:** `agent.errors.use((error, ctx, next) => { ... })`

## Context Helpers

`MessageContext` (`ctx`) provides: `sendTextReply()`, `sendMarkdownReply()`, `sendReaction()`, `getSenderAddress()`, `getClientAddress()`. Type guards: `ctx.isText()`, `ctx.isReply()`, `ctx.isReaction()`, etc. Conversation: `ctx.conversation.send()`, `ctx.conversation.sendMarkdown()`, `ctx.conversation.sendActions()`. State: `ctx.isAllowed`, `ctx.isDenied`, `ctx.isDm()`, `ctx.isGroup()`.

## Filters

```ts
import { filter } from "@xmtp/agent-sdk";
filter.fromSelf(message, client);  filter.hasContent(message);
filter.isDM(conversation);         filter.isGroup(conversation);
filter.isGroupAdmin(conversation, message);
```

## Conversations

```ts
await agent.createDmWithAddress("0x...");
await agent.createGroupWithAddresses(["0x...", "0x..."]);
```

## Common Mistakes

1. **Infinite loops** — Use specific events (`"text"`) not `"message"` without filtering
2. **Missing `await`** — All send methods are async
3. **No DB persistence** — `dbPath: null` loses state on restart; use `XMTP_DB_DIRECTORY` in production
4. **Installation limits** — Persist database to avoid creating new installations each restart

See api-reference.md for full types, attachment/transaction utilities, and env vars.
