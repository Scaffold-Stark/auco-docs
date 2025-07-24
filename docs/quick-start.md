---
sidebar_position: 2
---

# Quick Start

Get up and running with AUCO in minutes. This guide covers basic setup with different database options.

## Basic Example

Here's a minimal example to get you started:

```typescript
import { StarknetIndexer, LogLevel } from "auco";
import { abi as myContractAbi } from "./myContractAbi"; // Your contract ABI

const indexer = new StarknetIndexer({
  rpcNodeUrl: "https://starknet-sepolia-rpc.publicnode.com",
  wsNodeUrl: "wss://starknet-sepolia-rpc.publicnode.com",
  database: {
    type: "postgres",
    config: {
      connectionString: "postgresql://user:password@localhost:5432/mydb",
    },
  },
  logLevel: LogLevel.INFO,
});

// Register an event handler for ERC20 (STRK) Transfer
indexer.onEvent({
  contractAddress:
    "0x4718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d",
  abi: abi,
  eventName: "src::strk::erc20_lockable::ERC20Lockable::Transfer",
  handler: async (event) => {
    console.log("Received transfer event", event);
  },
});

// Start the indexer
indexer.start();
```

## Database Examples

Other than SQLite, you would have to have your database setup and readily accepting connections.

### PostgreSQL

<details>
<summary>PostgreSQL Setup</summary>

See [`example/index.ts`](https://github.com/Quantum3-Labs/auco/blob/main/example/index.ts) for a complete PostgreSQL example.

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  rpcNodeUrl: "https://starknet-sepolia-rpc.publicnode.com",
  wsNodeUrl: "wss://starknet-sepolia-rpc.publicnode.com",
  database: {
    type: "postgres",
    config: {
      connectionString: "postgresql://user:password@localhost:5432/mydb",
    },
  },
  logLevel: LogLevel.INFO,
});
```

</details>

### SQLite

<details>
<summary>SQLite Setup</summary>

Perfect for development and smaller applications:

```typescript
import Database from "better-sqlite3";
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  rpcNodeUrl: "https://starknet-sepolia-rpc.publicnode.com",
  wsNodeUrl: "wss://starknet-sepolia-rpc.publicnode.com",
  database: {
    type: "sqlite",
    config: {
      dbInstance: new Database("starknet_indexer.db"),
    },
  },
  logLevel: LogLevel.INFO,
  startingBlockNumber: "latest",
});
```

</details>

### MySQL

<details>
<summary>MySQL Setup</summary>

For MySQL databases:

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  rpcNodeUrl: "https://starknet-sepolia-rpc.publicnode.com",
  wsNodeUrl: "wss://starknet-sepolia-rpc.publicnode.com",
  database: {
    type: "mysql",
    config: {
      connectionString: "mysql://root:root@localhost:3306/starknet_indexer",
    },
  },
  logLevel: LogLevel.INFO,
  startingBlockNumber: "latest",
});
```

</details>

## Next Steps

- Use the [ABI Downloader CLI](./abi-downloader.md) to download contract ABIs
- Explore [API Reference](./api-reference/index.md)
- Handle [Chain Reorganizations](./api-reference/reorg-handling.md)
