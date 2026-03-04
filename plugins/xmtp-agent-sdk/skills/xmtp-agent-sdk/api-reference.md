# XMTP Agent SDK — API Reference

## Agent Class

```ts
class Agent<ContentTypes = unknown> extends EventEmitter<EventHandlerMap<ContentTypes>> {
  // Creation
  static create<ContentCodecs extends ContentCodec[] = []>(
    signer: Signer,
    options?: AgentCreateOptions<ContentCodecs>,
  ): Promise<Agent>;

  static createFromEnv<ContentCodecs extends ContentCodec[] = []>(
    options?: AgentCreateOptions<ContentCodecs>,
  ): Promise<Agent>;

  // Lifecycle
  start(options?: AgentStreamingOptions): Promise<void>;
  stop(): Promise<void>;

  // Middleware
  use(...middleware: Array<AgentMiddleware<ContentTypes> | AgentMiddleware<ContentTypes>[]>): this;
  errors: AgentErrorRegistrar<ContentTypes>;

  // Conversations
  createDmWithAddress(address: HexString, options?: CreateDmOptions): Promise<Dm>;
  createGroupWithAddresses(addresses: HexString[], options?: CreateGroupOptions): Promise<Group>;
  addMembersWithAddresses(group: Group, addresses: HexString[]): Promise<void>;
  getConversationContext(conversationId: string): Promise<ConversationContext | undefined>;

  // Properties
  client: Client<ContentTypes>;
  address: string | undefined;
  libxmtpVersion: string;
}
```

### AgentCreateOptions

```ts
type AgentCreateOptions<ContentCodecs extends ContentCodec[] = []> =
  Omit<ClientOptions & NetworkOptions, "codecs"> & {
    codecs?: ContentCodecs;
  };
```

Key options: `env` (`"local" | "dev" | "production" | "testnet" | "mainnet"`), `dbPath` (`string | null | ((inboxId: string) => string)`), `dbEncryptionKey`, `codecs`, `appVersion`, `loggingLevel`.

## Type Definitions

### Middleware Types

```ts
type AgentMiddleware<ContentTypes = unknown> = (
  ctx: MessageContext<unknown, ContentTypes>,
  next: () => Promise<void> | void,
) => Promise<void>;

type AgentErrorMiddleware<ContentTypes = unknown> = (
  error: unknown,
  ctx: AgentErrorContext<ContentTypes>,
  next: (err?: unknown) => Promise<void> | void,
) => Promise<void> | void;

type AgentErrorContext<ContentTypes = unknown> = Partial<AgentBaseContext<ContentTypes>> & {
  client: Client<ContentTypes>;
};
```

### Event Handler Map

```ts
type EventHandlerMap<ContentTypes> = {
  // Message events
  text:                   [ctx: MessageContext<string>];
  markdown:               [ctx: MessageContext<string>];
  reaction:               [ctx: MessageContext<Reaction>];
  reply:                  [ctx: MessageContext<EnrichedReply>];
  attachment:             [ctx: MessageContext<RemoteAttachment>];
  "inline-attachment":    [ctx: MessageContext<Attachment>];
  "multi-attachment":     [ctx: MessageContext<MultiRemoteAttachment>];
  actions:                [ctx: MessageContext<Actions>];
  intent:                 [ctx: MessageContext<Intent>];
  "read-receipt":         [ctx: MessageContext<ReadReceipt>];
  "transaction-reference":[ctx: MessageContext<TransactionReference>];
  "wallet-send-calls":    [ctx: MessageContext<WalletSendCalls>];
  "group-update":         [ctx: MessageContext<GroupUpdated>];
  "leave-request":        [ctx: MessageContext<LeaveRequest>];
  message:                [ctx: MessageContext<unknown>];
  unknownMessage:         [ctx: MessageContext<unknown>];

  // Conversation events
  conversation: [ctx: ConversationContext];
  dm:           [ctx: ConversationContext<ContentTypes, Dm>];
  group:        [ctx: ConversationContext<ContentTypes, Group>];

  // Lifecycle events
  start:          [ctx: ClientContext];
  stop:           [ctx: ClientContext];
  unhandledError: [error: Error];
};
```

## Context Classes

### ClientContext

Base context available in lifecycle events.

```ts
class ClientContext<ContentTypes = unknown> {
  client: Client<ContentTypes>;
  getClientAddress(): string | undefined;
}
```

### ConversationContext

Extends `ClientContext`. Available in conversation events and as base for message events.

```ts
class ConversationContext<ContentTypes, ConversationType extends Conversation> extends ClientContext<ContentTypes> {
  conversation: ConversationType;

  // Type guards
  isDm(): this is ConversationContext<ContentTypes, Dm>;
  isGroup(): this is ConversationContext<ContentTypes, Group>;

  // Sending
  sendRemoteAttachment(file: File, uploadCallback: AttachmentUploadCallback): Promise<void>;

  // Consent state
  isAllowed: boolean;
  isDenied: boolean;
  isUnknown: boolean;
}
```

### MessageContext

Extends `ConversationContext`. Available in all message event handlers.

```ts
class MessageContext<MessageContentType, ContentTypes> extends ConversationContext<ContentTypes> {
  message: DecodedMessageWithContent<MessageContentType>;

  // Type guards (narrow message content type)
  isText(): this is MessageContext<string>;
  isMarkdown(): this is MessageContext<string>;
  isReply(): this is MessageContext<Reply>;
  isReaction(): this is MessageContext<Reaction>;
  isReadReceipt(): this is MessageContext<ReadReceipt>;
  isRemoteAttachment(): this is MessageContext<RemoteAttachment>;
  isTransactionReference(): this is MessageContext<TransactionReference>;
  isWalletSendCalls(): this is MessageContext<WalletSendCalls>;
  usesCodec<T extends ContentCodec>(codecClass: new () => T): boolean;

  // Sending
  sendTextReply(text: string): Promise<void>;
  sendMarkdownReply(markdown: string): Promise<void>;
  sendReaction(content: string, schema?: ReactionSchema): Promise<void>;

  // Utilities
  getSenderAddress(): Promise<string | undefined>;
}
```

## Filters

```ts
import { filter } from "@xmtp/agent-sdk"; // alias: `f`

filter.fromSelf(message, client): boolean;
filter.hasContent(message): message is DecodedMessageWithContent;
filter.isDM(conversation): conversation is Dm;
filter.isGroup(conversation): conversation is Group;
filter.isGroupAdmin(conversation, message): boolean;
filter.isGroupSuperAdmin(conversation, message): boolean;
filter.usesCodec(message, CodecClass): boolean;
```

## CommandRouter

```ts
class CommandRouter<ContentTypes = unknown> {
  constructor(config?: { helpCommand?: `/${string}` });

  command(command: string, handler: AgentMessageHandler<string>): this;
  command(command: string, description: string, handler: AgentMessageHandler<string>): this;
  default(handler: AgentMessageHandler<string>): this;

  middleware(): AgentMiddleware<ContentTypes>;
  commandList: string[];
  handle(ctx: MessageContext<string>): Promise<boolean>;
}
```

Commands must start with `/`. When a command matches, `ctx.message.content` is set to the arguments (text after the command).

## PerformanceMonitor

```ts
interface PerformanceMonitorConfig {
  healthReportInterval?: number;       // ms between health reports (default: 60000, 0 to disable)
  criticalThresholdInterval?: number;  // ms threshold for slow response warning (default: 10000)
  onCriticalResponse?: (durationMs: number) => void;
  onHealthReport?: (report: HealthReport) => void;
  onResponse?: (durationMs: number) => void;
  onShutdown?: () => void;
}

interface HealthReport {
  cpuPercent: number;
  eventLoopDelayMs: number;
  heapMB: number;
  heapPercent: number;
  heapLimitMB: number;
  totalMB: number;
}

class PerformanceMonitor<ContentTypes = unknown> {
  constructor(config?: PerformanceMonitorConfig);
  middleware(): AgentMiddleware<ContentTypes>;
  shutdown(): void;
}
```

Register as the **first** middleware so it wraps all downstream processing.

## Attachment Utilities

```ts
type AttachmentUploadCallback = (attachment: EncryptedAttachment) => Promise<string>;

// Download and decrypt a remote attachment
function downloadRemoteAttachment(remoteAttachment: RemoteAttachment): Promise<Attachment>;

// Encrypt a file and create remote attachment metadata
function createRemoteAttachmentFromFile(
  file: File,
  uploadCallback: AttachmentUploadCallback,
): Promise<RemoteAttachment>;

// Create remote attachment from already-encrypted data
function createRemoteAttachment(
  encryptedAttachment: EncryptedAttachment,
  fileUrl: string,
): RemoteAttachment;
```

## Transaction Utilities

```ts
// Create ERC-20 token transfer request
function createERC20TransferCalls(options: {
  chain: Chain;
  tokenAddress: Hex;
  from: Hex;
  to: Hex;
  amount: bigint;
  description: string;
}): WalletSendCalls;

// Create native token transfer request (ETH, etc.)
function createNativeTransferCalls(options: {
  chain: Chain;
  from: Hex;
  to: Hex;
  amount: bigint;
  description: string;
}): WalletSendCalls;

// Read ERC-20 balance from blockchain
function getERC20Balance(options: {
  chain: Chain;
  tokenAddress: Hex;
  address: Hex;
  transport?: Transport;
}): Promise<bigint>;

// Read ERC-20 decimals from blockchain
function getERC20Decimals(options: {
  chain: Chain;
  tokenAddress: Hex;
  transport?: Transport;
}): Promise<number>;

// Minimal ERC-20 ABI (transfer, balanceOf, decimals)
const erc20Abi: readonly [...];
```

## User Utilities

```ts
type User = {
  key: Hex;
  account: PrivateKeyAccount;
  wallet: WalletClient;
};

// Create a user (random key if none provided)
function createUser(key?: HexString, chain?: Chain): User;

// Create an XMTP Signer from a User
function createSigner(user: User): Signer;

// Create an XMTP Identifier from a User
function createIdentifier(user: User): Identifier;

// Create an ENS/web3 name resolver (uses web3.bio API)
function createNameResolver(apiKey?: string): (name: string) => Promise<string | null>;
```

## Debug Utilities

```ts
// Get a test URL for your agent on xmtp.chat
function getTestUrl<ContentTypes>(client: Client<ContentTypes>): string;

// Log comprehensive agent details (address, inbox, installations, etc.)
function logDetails<ContentTypes>(agent: Agent<ContentTypes>): Promise<void>;

// Get installation info for the current client
function getInstallationInfo<ContentTypes>(client: Client<ContentTypes>): Promise<InstallationInfo>;
```

## Environment Variables

| Variable                 | Purpose                          | Example                             |
| ------------------------ | -------------------------------- | ----------------------------------- |
| `XMTP_WALLET_KEY`        | Private key (hex with 0x prefix) | `0x1234...abcd`                     |
| `XMTP_ENV`               | Network environment              | `dev`, `production`                 |
| `XMTP_DB_DIRECTORY`      | Database directory path          | `./data`                            |
| `XMTP_DB_ENCRYPTION_KEY` | Database encryption key (hex)    | `0xabcd...1234`                     |
| `XMTP_FORCE_DEBUG_LEVEL` | Force log level                  | `Off`, `Error`, `Warn`, `Info`, `Debug`, `Trace` |
| `XMTP_GATEWAY_HOST`      | Custom gateway URL               | `https://custom-gateway.example.com` |

## AgentError

```ts
class AgentError extends Error {
  code: number;
  cause?: unknown;
  constructor(code: number, message: string, cause?: unknown);
}

class AgentStreamingError extends AgentError {}
```

Error codes:
- `1000` — Invalid wallet key format
- `1001` — Conversation stream value error
- `1002` — Conversation streaming error
- `1003` — Conversation not found for message
- `1004` — Message streaming error
- `2000` — Name resolution failure
- `9999` — Unhandled error (default handler)
