---
sidebar_position: 1
---

# Introduction

[![CI](https://github.com/Quantum3-Labs/auco/actions/workflows/ci.yaml/badge.svg)](https://github.com/Quantum3-Labs/auco/actions/workflows/ci.yaml)
[![npm version](https://badge.fury.io/js/auco.svg)](https://badge.fury.io/js/auco)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Auco** is a high-performance TypeScript/Node.js indexer for Starknet events, supporting PostgreSQL, SQLite, MySQL, and real-time event handling via WebSocket and RPC.

## Key Features

- 🔄 **Real-time Event Indexing**: Listen to Starknet events from specified contracts
- 💾 **Data Persistence**: Store events and block data in PostgreSQL, SQLite, or MySQL
- 🔄 **Chain Reorg Handling**: Detect and handle blockchain reorganizations gracefully
- 🔌 **Extensible Architecture**: Register custom event handlers for any contract
- 🛡️ **Type Safety**: Full TypeScript support with type-safe event handling
- 🔄 **Retry Logic**: Automatic retry mechanisms with exponential backoff
- 📊 **Historical Processing**: Process historical blocks with seamless real-time transition
- ⚡ **High Performance**: Optimized for speed with parallel processing and efficient data handling

:::info Performance Benchmark
AUCO is **83% faster** than APIBARA in processing Starknet Transfer events, processing 159,885 events in 147.63s compared to APIBARA's 152,604 events in 270.50s (MacBook M1 Pro benchmark).
:::

## Installation

### From npm (Recommended)

```bash
npm install auco
```

### From source

<details>
<summary>Click to expand source installation</summary>

```bash
git clone https://github.com/Quantum3-Labs/auco.git
cd auco
npm install
npm run build
```

</details>

## Requirements

:::warning Prerequisites
Before using AUCO, ensure you have:
:::

- **Node.js**: Version 18 or higher
- **Database**: PostgreSQL 12+, SQLite, or MySQL
- **WebSocket endpoint**: Starknet node with WebSocket support (e.g., [Infura](https://infura.io/), local node with WS enabled)
- **Starknet node**: Spec version 0.8 or above

:::tip Getting a Starknet Node
For local development, see the quick setup guide in the [CONTRIBUTING.md](https://github.com/Quantum3-Labs/auco/blob/main/CONTRIBUTING.md#prerequisites).
:::
