---
sidebar_position: 3
---

# API Reference

Complete API documentation for AUCO's main classes and methods.

## StarknetIndexer Class

The main class for indexing Starknet events.

### Constructor

```typescript
new StarknetIndexer(config: IndexerConfig)
```

### Configuration

The `IndexerConfig` object accepts the following options:

| Option                | Type                 | Required | Description                                                   |
| --------------------- | -------------------- | -------- | ------------------------------------------------------------- |
| `rpcNodeUrl`          | `string`             | ✅       | Starknet RPC endpoint                                         |
| `wsNodeUrl`           | `string`             | ✅       | Starknet WebSocket endpoint                                   |
| `database`            | `DatabaseConfig`     | ✅       | Database configuration object with type and config properties |
| `logLevel`            | `LogLevel`           | ❌       | Log verbosity (DEBUG, INFO, WARN, ERROR)                      |
| `startingBlockNumber` | `number \| 'latest'` | ❌       | Starting block number for indexing                            |

#### Database Configuration

The `database` configuration supports multiple database types:

<details>
<summary>PostgreSQL Configuration</summary>

```typescript
{
  database: {
    type: "postgres",
    config: {
      connectionString: "postgresql://user:password@localhost:5432/mydb",
      // OR use individual options:
      host: "localhost",
      port: 5432,
      database: "mydb",
      user: "username",
      password: "password"
    }
  }
}
```

</details>

<details>
<summary>SQLite Configuration</summary>

```typescript
import Database from "better-sqlite3";

{
  database: {
    type: "sqlite",
    config: {
      dbInstance: new Database("path/to/database.db")
    }
  }
}
```

</details>

<details>
<summary>MySQL Configuration</summary>

```typescript
{
  database: {
    type: "mysql",
    config: {
      connectionString: "mysql://user:password@localhost:3306/mydb",
      // OR use individual options:
      host: "localhost",
      port: 3306,
      database: "mydb",
      user: "username",
      password: "password"
    }
  }
}
```

</details>

#### Log Levels

```typescript
enum LogLevel {
  DEBUG = "debug",
  INFO = "info",
  WARN = "warn",
  ERROR = "error",
}
```

## Core Methods

### indexer.start()

Starts the indexer and begins processing events.

```typescript
indexer.start(): Promise<void>
```

:::info
This method initializes the database connection, sets up WebSocket listeners, and begins processing historical blocks if a starting block is specified.
:::

**Example:**

```typescript
await indexer.start();
console.log("Indexer started successfully");
```

### indexer.stop()

Gracefully stops the indexer and closes all connections.

```typescript
indexer.stop(): Promise<void>
```

:::warning Graceful Shutdown
Always call `stop()` to ensure proper cleanup of database connections and WebSocket listeners.
:::

**Example:**

```typescript
process.on("SIGINT", async () => {
  console.log("Stopping indexer...");
  await indexer.stop();
  process.exit(0);
});
```

### indexer.onEvent()

Registers an event handler for a specific contract event.

```typescript
indexer.onEvent(config: EventHandlerConfig): void
```

#### EventHandlerConfig

| Property          | Type           | Required | Description                  |
| ----------------- | -------------- | -------- | ---------------------------- |
| `contractAddress` | `string`       | ✅       | Contract address to monitor  |
| `abi`             | `Abi`          | ✅       | Contract ABI                 |
| `eventName`       | `string`       | ✅       | Event name to listen for     |
| `handler`         | `EventHandler` | ✅       | Function to handle the event |

#### EventHandler Function

```typescript
type EventHandler = (
  event: ParsedEvent,
  client: DatabaseClient,
  indexer: StarknetIndexer,
) => Promise<void>;
```

**Parameters:**

- `event`: The parsed event data
- `client`: Database client for custom queries
- `indexer`: Reference to the indexer instance

**Example:**

```typescript
indexer.onEvent({
  contractAddress: "0x1234567890abcdef",
  abi: myContractAbi,
  eventName: "Transfer",
  handler: async (event, client, indexer) => {
    const { from_address, to_address, amount } = event.data;

    // Custom logic here
    console.log(`Transfer: ${amount} from ${from_address} to ${to_address}`);

    // Use database client for custom queries
    await client.query("INSERT INTO transfers (...) VALUES (...)", [
      from_address,
      to_address,
      amount,
    ]);
  },
});
```

:::tip Multiple Events
You can register multiple event handlers for different contracts and events by calling `onEvent()` multiple times.
:::

### indexer.onReorg()

Registers a handler for blockchain reorganizations.

```typescript
indexer.onReorg(config: ReorgHandlerConfig): void
```

#### ReorgHandlerConfig

| Property  | Type           | Required | Description                        |
| --------- | -------------- | -------- | ---------------------------------- |
| `handler` | `ReorgHandler` | ✅       | Function to handle reorganizations |

#### ReorgHandler Function

```typescript
type ReorgHandler = (forkedBlock: Block) => Promise<void>;
```

**Example:**

```typescript
indexer.onReorg({
  handler: async (forkedBlock) => {
    console.log(`Reorg detected at block ${forkedBlock.block_number}`);

    // Custom reorg handling logic
    // The indexer automatically rolls back non-canonical data
    // You should implement your own business logic here

    // Example: Clear cache or update external systems
    await clearCacheFromBlock(forkedBlock.block_number);
    await notifyExternalSystems(forkedBlock);
  },
});
```

:::warning Important
The indexer automatically handles database rollbacks for non-canonical blocks, but you must implement your own business logic for handling reorgs (e.g., clearing caches, updating external systems).
:::

## Types

### ParsedEvent

The event object passed to event handlers:

```typescript
interface ParsedEvent {
  data: Record<string, any>; // Decoded event data
  keys: string[]; // Event keys
  from_address: string; // Contract address
  block_hash: string; // Block hash
  block_number: number; // Block number
  transaction_hash: string; // Transaction hash
  event_index: number; // Event index in transaction
}
```

### Block

The block object passed to reorg handlers:

```typescript
interface Block {
  block_number: number;
  block_hash: string;
  parent_hash: string;
  timestamp: number;
  // ... other block properties
}
```
