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
  rpcNodeUrl: "https://starknet-mainnet.infura.io/v3/YOUR_KEY",
  wsNodeUrl: "wss://starknet-mainnet.infura.io/ws/v3/YOUR_KEY",
  database: {
    type: "postgres",
    config: {
      connectionString: "postgresql://user:password@localhost:5432/mydb",
    },
  },
  logLevel: LogLevel.INFO,
});

// Register an event handler
indexer.onEvent({
  contractAddress: "0x...", // Your contract address
  abi: myContractAbi,
  eventName: "Transfer",
  handler: async (event, client, indexer) => {
    console.log("Received event:", event);
    // Your custom logic here
  },
});

// Start the indexer
indexer.start();
```

## Database Examples

### PostgreSQL

<details>
<summary>PostgreSQL Setup</summary>

See [`example/index.ts`](https://github.com/Quantum3-Labs/auco/blob/main/example/index.ts) for a complete PostgreSQL example.

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  rpcNodeUrl: "https://starknet-mainnet.infura.io/v3/YOUR_KEY",
  wsNodeUrl: "wss://starknet-mainnet.infura.io/ws/v3/YOUR_KEY",
  database: {
    type: "postgres",
    config: {
      connectionString: "postgresql://user:password@localhost:5432/mydb",
    },
  },
  logLevel: LogLevel.INFO,
});
```

**Run the example:**

```bash
npx ts-node example/index.ts
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

**Run the example:**

```bash
npx ts-node example/sqlite.ts
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

**Run the example:**

```bash
npx ts-node example/mysql.ts
```

</details>

## ABI Downloader CLI

:::tip ABI Management
AUCO includes a built-in CLI to download contract ABIs directly from Starknet nodes, making it easy to generate TypeScript ABI files for your project.
:::

### Download Single Contract ABI

```bash
npx ts-node src/scripts/download-abi.ts single \
  --rpc-url <STARKNET_RPC_URL> \
  --address <CONTRACT_ADDRESS> \
  --name <CONTRACT_NAME> \
  [--output <OUTPUT_DIR>]
```

**Example:**

```bash
npx ts-node src/scripts/download-abi.ts single \
  --rpc-url https://starknet-mainnet.infura.io/v3/YOUR_KEY \
  --address 0x123...abc \
  --name MyContract
```

### Batch Download ABIs

<details>
<summary>Batch Mode Instructions</summary>

1. Create a JSON file (e.g., `contracts.json`):

```json
{
  "MyContract": "0x123...abc",
  "AnotherContract": "0x456...def"
}
```

2. Run the batch command:

```bash
npx ts-node src/scripts/download-abi.ts batch \
  --rpc-url <STARKNET_RPC_URL> \
  --file contracts.json \
  [--output <OUTPUT_DIR>]
```

The script generates TypeScript ABI files in the output directory and creates an `index.ts` exporting all ABIs.

</details>

## Next Steps

- Learn about [Configuration Options](./configuration/index.md)
- Explore [API Reference](./api-reference/index.md)
- Set up [Database Configuration](./configuration/database.md)
- Handle [Chain Reorganizations](./api-reference/reorg-handling.md)
