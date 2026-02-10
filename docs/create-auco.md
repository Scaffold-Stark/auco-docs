---
sidebar_position: 4
---

# Create Auco (Project Scaffolding CLI)

Use the **`create-auco`** CLI to scaffold a fully‑configured Starknet indexer project powered by Auco.

It sets up:

- A TypeScript project with a ready‑to‑run indexer and Express API.
- PostgreSQL via `docker-compose`.
- `.env` / `.env.example` with sensible defaults for Starknet Sepolia.
- An example STRK ERC‑20 Transfer indexer and API endpoints.

## Usage

The CLI is published as `create-auco`

```bash
npm create auco my-starknet-indexer
# or
npx create-auco my-starknet-indexer
```

### Prompts & Options

If you do **not** pass `--yes`, the CLI will prompt for:

- **Project name** – defaults to `my-starknet-indexer` if left empty.

Additional behavior:

- Project names must match `^[a-zA-Z0-9-_]+$` (letters, numbers, hyphens, underscores).
- If the target directory exists and is **not empty**, the CLI aborts with an error.

## What `create-auco` Generates

Given a project name like `my-starknet-indexer`, the CLI creates:

```text
my-starknet-indexer/
├── src/
│   ├── index.ts          # Main indexer + Express API server
│   └── abi/
│       └── StrkToken.ts  # Example STRK token ABI
├── .env                  # Default environment variables
├── .env.example          # Example env file for sharing
├── .gitignore
├── docker-compose.yml    # PostgreSQL service
├── package.json
├── tsconfig.json
└── README.md             # Project-specific README
```

Key generated files and their roles:

- **`package.json`**
  - Scripts:
    - `dev`: `npx ts-node src/index.ts`
    - `build`: `tsc`
    - `start`: `node dist/index.js`
    - `db:up`: `docker compose up -d`
    - `db:down`: `docker compose down`
    - `db:reset`: `docker compose down -v && docker compose up -d`
    - `download-abi`: `npx auco-download-abi`
  - Dependencies:
    - `auco` (indexer library)
    - `express`
    - `dotenv`
  - Dev dependencies:
    - `typescript`, `ts-node`, `@types/node`, `@types/express`

- **`tsconfig.json`**
  - Configured for:
    - `rootDir: ./src`
    - `outDir: ./dist`
    - `strict` TypeScript options.

- **`.env` / `.env.example`**
  - Prepopulated with:
    - `RPC_URL`, `WS_URL` for Starknet Sepolia public RPC.
    - `DATABASE_URL` pointing to local PostgreSQL.
    - `PORT` for the API server (default `3000`).
    - `STARTING_BLOCK` (e.g. `latest`) and `LOG_LEVEL` (e.g. `info`).

- **`docker-compose.yml`**
  - Spins up a PostgreSQL 16‑alpine instance:
    - Exposes port `5432`.
    - Uses database name `starknet_indexer`.
    - Includes a basic healthcheck.

- **`src/index.ts`**
  - Boots an `StarknetIndexer` instance configured from environment variables.
  - Registers a handler for STRK **Transfer** events on Starknet Sepolia.
  - Stores recent transfers in memory.
  - Starts an Express server with endpoints:
    - `GET /health` – indexer and service health.
    - `GET /api/transfers` – recent transfers (`?limit=`).
    - `GET /api/transfers/:txHash` – transfer by transaction hash.
    - `GET /api/address/:address/transfers` – transfers for an address.

- **`src/abi/StrkToken.ts`**
  - Example TypeScript ABI for the STRK token, with typed event definitions.

## After Scaffolding: Next Steps

Inside your new project directory:

```bash
cd my-starknet-indexer
npm install

# Start PostgreSQL
npm run db:up

# Copy and customize environment variables
cp .env.example .env
# edit .env as needed (RPC_URL, DATABASE_URL, etc.)

# Run the indexer in development mode
npm run dev
```

Once running, you should have:

- API at `http://localhost:3000`.
- Health check at `http://localhost:3000/health`.
- Transfers API at `http://localhost:3000/api/transfers`.

## Customizing Your Indexer

To adapt the generated project to your own contracts:

1. **Provide your own ABI**
   - Use the [ABI Downloader CLI](./abi-downloader.md) to generate strongly‑typed ABIs, or
   - Add ABI files under `src/abi/` manually.

2. **Register new event handlers**
   - Edit `src/index.ts` and add additional `indexer.onEvent` registrations for your contracts/events.

3. **Persist data in your database**
   - Replace the in‑memory storage in `src/index.ts` with real database writes using the provided DB handler.

This workflow gives you a working, batteries‑included Starknet indexer in minutes, ready to customize for your own contracts and data models.



