---
sidebar_position: 3
---

# ABI Downloader CLI

:::tip ABI Management
AUCO includes a built-in CLI to download contract ABIs directly from Starknet nodes, making it easy to generate TypeScript ABI files for your project.
:::

## Download Single Contract ABI

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
  --rpc-url https://starknet-sepolia-rpc.publicnode.com \
  --address 0x123...abc \
  --name MyContract
```

## Batch Download ABIs

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

## Options

### Single Mode Options

- `--rpc-url`: Starknet RPC endpoint URL
- `--address`: Contract address to download ABI for
- `--name`: Name for the generated TypeScript file
- `--output`: Output directory (optional, defaults to current directory)

### Batch Mode Options

- `--rpc-url`: Starknet RPC endpoint URL
- `--file`: JSON file containing contract names and addresses
- `--output`: Output directory (optional, defaults to current directory)

## Output

The CLI generates:

- Individual TypeScript files for each contract ABI
- An `index.ts` file that exports all downloaded ABIs
- Properly typed TypeScript definitions for use in your AUCO project
