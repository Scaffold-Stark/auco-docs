---
sidebar_position: 1
---

# Configuration Overview

Configure AUCO to match your specific requirements and environment.

## Configuration Sections

### [Database Configuration](./database.md)

Set up your preferred database backend:

- **PostgreSQL**: Production-ready with advanced features
- **SQLite**: Perfect for development and smaller applications
- **MySQL**: Alternative SQL database option

### Core Configuration Options

All configuration is done through the `IndexerConfig` object passed to the `StarknetIndexer` constructor.

#### Required Options

| Option       | Type     | Description                     |
| ------------ | -------- | ------------------------------- |
| `rpcNodeUrl` | `string` | Starknet RPC endpoint URL       |
| `wsNodeUrl`  | `string` | Starknet WebSocket endpoint URL |
| `database`   | `object` | Database configuration object   |

#### Optional Options

| Option                | Type                 | Default  | Description                         |
| --------------------- | -------------------- | -------- | ----------------------------------- |
| `logLevel`            | `LogLevel`           | `INFO`   | Logging verbosity level             |
| `startingBlockNumber` | `number \| 'latest'` | `latest` | Block number to start indexing from |

## Example Configuration

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  // Required: Node endpoints
  rpcNodeUrl: "https://starknet-mainnet.infura.io/v3/YOUR_KEY",
  wsNodeUrl: "wss://starknet-mainnet.infura.io/ws/v3/YOUR_KEY",

  // Required: Database configuration
  database: {
    type: "postgres",
    config: {
      connectionString: "postgresql://user:pass@localhost:5432/db",
    },
  },

  // Optional: Logging level
  logLevel: LogLevel.INFO,

  // Optional: Starting block (defaults to 'latest')
  startingBlockNumber: 1000000, // Start from specific block
  // OR
  // startingBlockNumber: "latest", // Start from latest block
});
```

## Environment-Specific Configurations

### Development

```typescript
const config = {
  rpcNodeUrl: "https://starknet-sepolia-rpc.publicnode.com",
  wsNodeUrl: "wss://starknet-sepolia-rpc.publicnode.com",
  database: {
    type: "sqlite",
    config: {
      dbInstance: new Database("dev.db"),
    },
  },
  logLevel: LogLevel.DEBUG,
  startingBlockNumber: "latest",
};
```

### Production

```typescript
const config = {
  rpcNodeUrl: process.env.STARKNET_RPC_URL,
  wsNodeUrl: process.env.STARKNET_WS_URL,
  database: {
    type: "postgres",
    config: {
      connectionString: process.env.DATABASE_URL,
      ssl: true,
      max: 20,
    },
  },
  logLevel: LogLevel.WARN,
  startingBlockNumber: parseInt(process.env.START_BLOCK) || "latest",
};
```

## Configuration Validation

AUCO validates your configuration on startup and will throw descriptive errors for invalid configurations:

```typescript
// This will throw an error
const invalidConfig = {
  rpcNodeUrl: "invalid-url", // ❌ Invalid URL format
  // ❌ Missing wsNodeUrl
  // ❌ Missing database config
};
```

:::tip Environment Variables
Consider using environment variables for sensitive configuration like database credentials and API keys.
:::

## Next Steps

- Set up your [Database Configuration](./database.md)
- Explore the [API Reference](../api-reference/index.md)
- Check the [Troubleshooting Guide](../troubleshooting.md) for common issues
