[![CI](https://github.com/Quantum3-Labs/auco/actions/workflows/ci.yaml/badge.svg)](https://github.com/Quantum3-Labs/auco/actions/workflows/ci.yaml)
[![npm version](https://badge.fury.io/js/auco.svg)](https://badge.fury.io/js/auco)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

# Auco

A TypeScript/Node.js indexer for Starknet events, supporting PostgreSQL, SQLite, and real-time event handling via WebSocket and RPC.

## Features

- 🔄 **Real-time Event Indexing**: Listen to Starknet events from specified contracts
- 💾 **Data Persistence**: Store events and block data in PostgreSQL
- 🔄 **Chain Reorg Handling**: Detect and handle blockchain reorganizations gracefully
- 🔌 **Extensible Architecture**: Register custom event handlers for any contract
- 🛡️ **Type Safety**: Full TypeScript support with type-safe event handling
- 🔄 **Retry Logic**: Automatic retry mechanisms with exponential backoff
- 📊 **Historical Processing**: Process historical blocks with seamless real-time transition
- 🔌 **Multiple DB Support**: Supports PostgreSQL, SQLite, and MySQL out of the box.
- ⚡ **High Performance**: Optimized for speed with parallel processing and efficient data handling

## Installation

### From npm (Recommended)

```bash
npm install auco
```

### From source

```bash
git clone https://github.com/Quantum3-Labs/auco.git
cd auco
npm install
npm run build
```

## Requirements

- **Node.js**: Version 18 or higher
- **PostgreSQL**: Version 12 or higher (can be switched with SQLite or MySQL)
- **WebSocket endpoint**: Starknet node with WebSocket support (e.g., Infura, local node with WS enabled)
- **Starknet node spec version 0.8 or above**: Compatible with Starknet nodes running spec version 0.8+
- **How to install a Starknet node?** See the quick guide in [CONTRIBUTING.md](CONTRIBUTING.md#prerequisites).

## Quick Start

### PostgreSQL Example

See [`example/index.ts`](./example/index.ts) for a PostgreSQL-based example.

```typescript
import { StarknetIndexer, LogLevel } from "auco";
import { abi as myContractAbi } from "./myContractAbi"; // Provide your contract ABI

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

indexer.onEvent({
  contractAddress: "0x...",
  abi: myContractAbi,
  eventName: "Transfer",
  handler: async (event, client, indexer) => {
    console.log("Received event:", event);
    // Custom logic here
  },
});

indexer.start();
```

### SQLite Example

1. **Configure your environment**:

   - Edit `example/sqlite.ts` with your node URLs and contract address
   - Update your database config to SQLite

   ```typescript
   import Database from "better-sqlite3";

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

2. **Run the example**:
   ```bash
   npx ts-node example/sqlite.ts
   ```

### MySQL Example

1. **Configure your environment**:

   - Edit `example/mysql.ts` with your node URLs and contract address
   - Update your database config to MySQL

   ```typescript
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

2. **Run the example**:
   ```bash
   npx ts-node example/mysql.ts
   ```

## ABI Downloader CLI

You can download contract ABIs directly from Starknet nodes using the built-in CLI script. This is useful for generating TypeScript ABI files for your project.

### Usage

Run the script with `npx` (recommended) or directly with `ts-node`/`node`:

#### Single Contract

Download the ABI for a single contract:

```bash
npx ts-node src/scripts/download-abi.ts single \
  --rpc-url <STARKNET_RPC_URL> \
  --address <CONTRACT_ADDRESS> \
  --name <CONTRACT_NAME> \
  [--output <OUTPUT_DIR>]
```

- `--rpc-url` (required): Starknet RPC endpoint URL
- `--address` (required): Contract address
- `--name` (required): Name for the generated ABI file (no extension)
- `--output` (optional): Output directory (default: `generated/abis`)

**Example:**

```bash
npx ts-node src/scripts/download-abi.ts single \
  --rpc-url https://starknet-mainnet.infura.io/v3/YOUR_KEY \
  --address 0x123...abc \
  --name MyContract
```

#### Batch Mode

Download ABIs for multiple contracts defined in a JSON file:

1. Create a JSON file (e.g., `contracts.json`) with the following format:

   ```json
   {
     "MyContract": "0x123...abc",
     "AnotherContract": "0x456...def"
   }
   ```

2. Run:
   ```bash
   npx ts-node src/scripts/download-abi.ts batch \
     --rpc-url <STARKNET_RPC_URL> \
     --file contracts.json \
     [--output <OUTPUT_DIR>]
   ```

- `--file` (required): Path to the JSON file mapping contract names to addresses
- Other options as above

**Example:**

```bash
npx ts-node src/scripts/download-abi.ts batch \
  --rpc-url https://starknet-mainnet.infura.io/v3/YOUR_KEY \
  --file contracts.json
```

The script will generate TypeScript ABI files in the specified output directory and create an `index.ts` exporting all ABIs.

---

## Performance

AUCO is designed for high-performance event indexing with optimized parallel processing and efficient data handling. Our benchmarks demonstrate significant performance advantages:

### Benchmark Results: Transfer Events on Starknet Mainnet

| Metric               | AUCO    | APIBARA | Improvement          |
| -------------------- | ------- | ------- | -------------------- |
| **Sync Time**        | 147.63s | 270.50s | **83% faster**       |
| **Events Processed** | 159,885 | 152,604 | **4.8% more events** |

**Benchmark Parameters:**

- **Block Range**: 1,500,000 to 1,505,000 (5,000 blocks)
- **Event Type**: Starknet Transfer events
- **Total Events**: ~150,000 events processed
- **Device**: MacBook M1 Pro (16GB RAM)

## Configuration

| Option                | Type                 | Required | Description                                                   |
| --------------------- | -------------------- | -------- | ------------------------------------------------------------- |
| `rpcNodeUrl`          | `string`             | ✅       | Starknet RPC endpoint                                         |
| `wsNodeUrl`           | `string`             | ✅       | Starknet WebSocket endpoint                                   |
| `database`            | `object`             | ✅       | Database configuration object with type and config properties |
| `logLevel`            | `LogLevel`           | ❌       | Log verbosity (DEBUG, INFO, WARN, ERROR)                      |
| `startingBlockNumber` | `number \| 'latest'` | ❌       | Starting block number for indexing                            |

## Handling Chain Reorgs

The indexer automatically detects and handles chain reorganizations:

```typescript
indexer.onReorg({
  handler: async (forkedBlock) => {
    console.log(`Reorg detected at block ${forkedBlock.block_number}`);
    // Implement your reorg handling logic
    // The indexer automatically rolls back non-canonical data
  },
});
```

**Important**: You must implement your own business logic for handling reorgs. The indexer provides the hook and basic database cleanup.

## Development

### Running Tests

```bash
npm test
```

### Code Quality

```bash
npm run lint          # Check code style
npm run lint:fix      # Fix code style issues
npm run format        # Format code with Prettier
```

### Building

```bash
npm run build         # Compile TypeScript
```

## Roadmap

### 📋 Planned

- [x] Additional database (MongoDB, MySQL, etc.) ✅
- [ ] Built-in monitoring and health checks
- [ ] Advanced caching layer
- [ ] Docker containerization
- [ ] Event filtering by multiple criteria
- [ ] Instant event processing via WebSocket

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Development Setup

1. **Fork and clone** the repository
2. **Install dependencies**:

   ```bash
   npm install
   ```

3. **Setup development environment**:

   ```bash
   # Start local PostgreSQL (if using Docker)
   docker run --name postgres-dev -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:13

   # Start local Starknet devnet
   npm run chain
   ```

4. **Run tests**:

   ```bash
   npm test
   ```

5. **Submit a pull request** with your changes

### Code Style

- Use TypeScript for all new code
- Follow the existing code style (enforced by ESLint and Prettier)
- Write tests for new functionality
- Update documentation as needed

## Troubleshooting

### Common Issues

| Issue                        | Solution                                        |
| ---------------------------- | ----------------------------------------------- |
| PostgreSQL connection failed | Ensure PostgreSQL is running and accessible     |
| WebSocket connection failed  | Verify your node supports WebSocket connections |
| ABI parsing errors           | Check your ABI file and event names             |
| Memory usage high            | Consider processing events in smaller batches   |

### Getting Help

- 📖 **Documentation**: Check the [API Reference](docs/API.md)
- 🐛 **Issues**: Report bugs on [GitHub Issues](https://github.com/Quantum3-Labs/auco/issues)
- 💬 **Discussions**: Join our [GitHub Discussions](https://github.com/Quantum3-Labs/auco/discussions)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built with [Starknet.js](https://github.com/0xs34n/starknet.js)
- Type-safe ABI parsing with [abi-wan-kanabi](https://github.com/keep-starknet-strange/abi-wan-kanabi)
- Database operations with [node-postgres](https://github.com/brianc/node-postgres), [mysql2](https://github.com/sidorares/node-mysql2), and [better-sqlite3](https://github.com/WiseLibs/better-sqlite3)
