---
name: xmtp-agent-sdk
description: Use when building, extending, or modifying XMTP messaging agents using @xmtp/agent-sdk, or when working with XMTP event handlers, middleware, message content types, or agent lifecycle
---

# XMTP Agent SDK

Build event-driven, middleware-powered messaging agents on the XMTP network using `@xmtp/agent-sdk`.

## When to Use

- Creating a new XMTP agent from scratch
- Adding event handlers for message types (text, reactions, replies, attachments, etc.)
- Writing middleware (routing, filtering, monitoring)
- Working with message context helpers (replies, reactions, markdown)
- Handling conversations (DMs, groups)
- Sending attachments or transaction requests

## Quick Start

### Option 1: Explicit signer

```ts
import { Agent, createUser, createSigner, getTestUrl } from "@xmtp/agent-sdk";

const user = createUser();
const signer = createSigner(user);

const agent = await Agent.create(signer, {
  env: "dev",
  dbPath: null, // in-memory; provide a path to persist
});

agent.on("text", async (ctx) => {
  await ctx.sendTextReply("Hello from my agent!");
});

agent.on("start", (ctx) => {
  console.log(`Online: ${getTestUrl(ctx.client)}`);
});

await agent.start();
```

### Option 2: Environment variables

Set `XMTP_WALLET_KEY`, `XMTP_ENV`, and optionally `XMTP_DB_DIRECTORY` and `XMTP_DB_ENCRYPTION_KEY` in a `.env` file.

```ts
process.loadEnvFile(".env");

const agent = await Agent.createFromEnv();

agent.on("text", async (ctx) => {
  await ctx.sendTextReply("Hello!");
});

await agent.start();
```

## Event System

Register handlers with `agent.on(event, handler)`. The agent auto-filters self-messages and undefined content.

**Message events:** `text`, `markdown`, `reaction`, `reply`, `attachment`, `inline-attachment`, `multi-attachment`, `actions`, `intent`, `read-receipt`, `transaction-reference`, `wallet-send-calls`, `group-update`, `leave-request`, `message` (all messages), `unknownMessage`

**Conversation events:** `conversation` (any new), `dm` (new DM), `group` (new group)

**Lifecycle events:** `start`, `stop`, `unhandledError`

```ts
agent.on("text", async (ctx) => {
  await ctx.sendTextReply(`Echo: ${ctx.message.content}`);
});

agent.on("reaction", (ctx) => {
  console.log(`Reaction: ${ctx.message.content.content}`);
});

agent.on("dm", async (ctx) => {
  await ctx.conversation.send("Welcome to our DM!");
});

agent.on("group", async (ctx) => {
  await ctx.conversation.sendMarkdown("**Hello group!**");
});
```

**Warning:** The `"message"` event fires for every message. Always filter to prevent infinite loops:

```ts
import { filter } from "@xmtp/agent-sdk";

agent.on("message", async (ctx) => {
  if (filter.isText(ctx.message)) {
    await ctx.conversation.send(`Echo: ${ctx.message.content}`);
  }
});
```

## Middleware

Middleware functions receive `(ctx, next)`. Call `next()` to continue, `return` to stop, or `throw` to trigger error middleware.

```ts
import { Agent, type AgentMiddleware, filter } from "@xmtp/agent-sdk";

const onlyText: AgentMiddleware = async (ctx, next) => {
  if (filter.isText(ctx.message)) {
    await next();
  }
  return;
};

agent.use(onlyText);
```

### CommandRouter

Built-in slash-command routing middleware:

```ts
import { CommandRouter } from "@xmtp/agent-sdk";

const router = new CommandRouter({ helpCommand: "/help" });

router.command("/ping", "Check if agent is alive", async (ctx) => {
  await ctx.conversation.sendText("Pong!");
});

router.default(async (ctx) => {
  await ctx.conversation.sendText(`Unknown: ${ctx.message.content}`);
});

agent.use(router.middleware());
```

### Error Middleware

```ts
import { type AgentErrorMiddleware } from "@xmtp/agent-sdk";

const errorHandler: AgentErrorMiddleware = async (error, ctx, next) => {
  console.error("Error:", error);
  await next(error); // forward to next handler
};

agent.errors.use(errorHandler);

agent.on("unhandledError", (error) => {
  console.error("Unhandled:", error);
});
```

### PerformanceMonitor

```ts
import { PerformanceMonitor } from "@xmtp/agent-sdk";

const monitor = new PerformanceMonitor({
  healthReportInterval: 60_000,
  criticalThresholdInterval: 10_000,
  onResponse: (duration) => console.log(`Processed in ${duration}ms`),
});

agent.use(monitor.middleware());

agent.on("stop", () => monitor.shutdown());
```

## Message Context Helpers

Every message handler receives a `MessageContext` (`ctx`) with:

```ts
agent.on("text", async (ctx) => {
  // Sending
  await ctx.sendTextReply("Reply text");
  await ctx.sendMarkdownReply("**bold** reply");
  await ctx.sendReaction("👍");

  // Reading
  const content = ctx.message.content;        // typed per event
  const sender = await ctx.getSenderAddress(); // Ethereum address
  const myAddr = ctx.getClientAddress();       // agent's address

  // Type guards
  if (ctx.isText()) { /* ctx.message.content is string */ }
  if (ctx.isReply()) { /* ctx.message.content is EnrichedReply */ }
  if (ctx.isReaction()) { /* ctx.message.content is Reaction */ }

  // Conversation access
  await ctx.conversation.send("Direct send");
  await ctx.conversation.sendMarkdown("**markdown**");
  await ctx.conversation.sendActions({
    id: "action-1",
    description: "Choose:",
    actions: [{ id: "yes", label: "Yes" }, { id: "no", label: "No" }],
  });

  // Conversation type checks
  if (ctx.isDm()) { /* DM-specific logic */ }
  if (ctx.isGroup()) { /* Group-specific logic */ }

  // Consent state
  ctx.isAllowed;  // ConsentState.Allowed
  ctx.isDenied;   // ConsentState.Denied
  ctx.isUnknown;  // ConsentState.Unknown
});
```

## Filters

```ts
import { filter } from "@xmtp/agent-sdk"; // also importable as `f`

filter.fromSelf(message, client);           // true if sent by this agent
filter.hasContent(message);                 // true if content is defined
filter.isDM(conversation);                  // true if DM
filter.isGroup(conversation);               // true if Group
filter.isGroupAdmin(conversation, message); // true if sender is admin
filter.isGroupSuperAdmin(conversation, message);
filter.usesCodec(message, MyCodecClass);    // true if message uses codec
```

## Starting Conversations

```ts
const dm = await agent.createDmWithAddress("0x123...");
await dm.send("Hello!");

const group = await agent.createGroupWithAddresses(["0x123...", "0x456..."]);
await group.send("Hello group!");
```

## Attachments

```ts
import {
  downloadRemoteAttachment,
  type AttachmentUploadCallback,
} from "@xmtp/agent-sdk";

// Send
const uploadCallback: AttachmentUploadCallback = async (encrypted) => {
  // Upload encrypted.content.payload to your storage, return URL
  return "https://storage.example.com/file.enc";
};
await ctx.sendRemoteAttachment(file, uploadCallback);

// Receive
agent.on("attachment", async (ctx) => {
  const attachment = await downloadRemoteAttachment(ctx.message.content);
  console.log(attachment.filename);
});
```

## Transactions

```ts
import {
  createERC20TransferCalls,
  createNativeTransferCalls,
  getERC20Balance,
  getERC20Decimals,
} from "@xmtp/agent-sdk";
import { base } from "viem/chains";

// Query balance
const balance = await getERC20Balance({
  chain: base,
  tokenAddress: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  address: "0x123...",
});

// Create ERC-20 transfer request
const calls = createERC20TransferCalls({
  chain: base,
  tokenAddress: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  from: "0xSender",
  to: "0xReceiver",
  amount: 1_000_000n, // 1 USDC (6 decimals)
  description: "Transfer 1 USDC",
});
await ctx.conversation.sendWalletSendCalls(calls);

// Listen for transaction confirmations
agent.on("transaction-reference", async (ctx) => {
  const { networkId, reference } = ctx.message.content;
  console.log(`TX ${reference} on network ${networkId}`);
});
```

## User Utilities

```ts
import { createUser, createSigner, createNameResolver } from "@xmtp/agent-sdk";

// Create user with random key
const user = createUser();
// Or with existing private key
const user2 = createUser("0xYourPrivateKey");

const signer = createSigner(user);

// ENS/web3 name resolution (uses web3.bio)
const resolver = createNameResolver("optional-web3bio-api-key");
const address = await resolver("vitalik.eth");
```

## Common Mistakes

1. **Infinite message loops** — The `"message"` event fires for all messages. If your handler sends a message without filtering, it triggers itself. Use specific events like `"text"` instead, or filter with `filter.isText()`.

2. **Missing `await`** — All `ctx.send*` and `ctx.conversation.send*` methods are async. Forgetting `await` causes silent failures.

3. **Not persisting the database** — Using `dbPath: null` is fine for development but loses all state on restart. In production, provide a path or use `XMTP_DB_DIRECTORY`.

4. **Installation limits** — Each agent identity has a limited number of installations. Persist your database to avoid creating new installations on every restart.

See api-reference.md for full type signatures.
