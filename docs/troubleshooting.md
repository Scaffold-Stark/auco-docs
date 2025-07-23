---
sidebar_position: 4
---

# Troubleshooting

Common issues and solutions when working with AUCO.

## Common Issues

### Database Connection Issues

| Issue                            | Symptoms                        | Solution                                                          |
| -------------------------------- | ------------------------------- | ----------------------------------------------------------------- |
| **PostgreSQL connection failed** | `Connection refused` errors     | Ensure PostgreSQL is running and accessible on the specified port |
| **Authentication failed**        | `Authentication failed` errors  | Check username, password, and database permissions                |
| **Database does not exist**      | `Database "xyz" does not exist` | Create the database first or check the database name              |
| **SSL connection errors**        | SSL-related error messages      | Configure SSL properly or disable SSL for development             |

#### Database Connection Debugging

Enable debug logging to diagnose connection issues:

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  logLevel: LogLevel.DEBUG, // Enable detailed logging
  // ... rest of config
});
```

### Network and Node Issues

| Issue                           | Symptoms                                  | Solution                                                               |
| ------------------------------- | ----------------------------------------- | ---------------------------------------------------------------------- |
| **WebSocket connection failed** | `WebSocket connection failed` errors      | Verify your node supports WebSocket connections and the URL is correct |
| **RPC endpoint unreachable**    | `Network timeout` or `Connection refused` | Check if the RPC endpoint is accessible and not rate-limited           |
| **Invalid node response**       | Parsing errors or unexpected responses    | Ensure your node is running Starknet spec version 0.8 or above         |

#### Network Debugging

Test your endpoints manually:

```bash
# Test RPC endpoint
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"starknet_chainId","params":[],"id":1}' \
  YOUR_RPC_URL

# Test WebSocket (requires wscat)
npm install -g wscat
wscat -c YOUR_WEBSOCKET_URL
```

### ABI and Contract Issues

| Issue                       | Symptoms                                 | Solution                                             |
| --------------------------- | ---------------------------------------- | ---------------------------------------------------- |
| **ABI parsing errors**      | `Invalid ABI format` or parsing failures | Verify your ABI file format and event names          |
| **Event not found**         | `Event "EventName" not found in ABI`     | Check that the event name exactly matches the ABI    |
| **Contract not responding** | No events received despite activity      | Verify the contract address and ensure it's deployed |

#### ABI Debugging

Validate your ABI and contract setup:

```typescript
// Log ABI events to verify structure
console.log(
  "Available events:",
  abi.filter((item) => item.type === "event"),
);

// Test contract address format
console.log("Contract address:", contractAddress);
console.log("Is valid hex:", /^0x[a-fA-F0-9]+$/.test(contractAddress));
```

### Performance Issues

| Issue                        | Symptoms                        | Solution                                                                   |
| ---------------------------- | ------------------------------- | -------------------------------------------------------------------------- |
| **High memory usage**        | Process consuming excessive RAM | Consider processing events in smaller batches or optimizing event handlers |
| **Slow event processing**    | Events processing slowly        | Check database performance, add proper indexes, or optimize queries        |
| **WebSocket disconnections** | Frequent connection drops       | Implement reconnection logic or check network stability                    |

#### Performance Optimization

Monitor and optimize performance:

```typescript
// Add performance monitoring to event handlers
indexer.onEvent({
  contractAddress: "0x...",
  abi: myAbi,
  eventName: "Transfer",
  handler: async (event, client, indexer) => {
    const start = Date.now();

    // Your event processing logic
    await processEvent(event);

    const duration = Date.now() - start;
    if (duration > 1000) {
      // Log slow operations
      console.warn(`Slow event processing: ${duration}ms`);
    }
  },
});
```

## Error Messages and Solutions

### Common Error Patterns

#### "Cannot connect to database"

```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Solution**: Ensure your database server is running:

```bash
# PostgreSQL
brew services start postgresql  # macOS
sudo service postgresql start  # Linux

# MySQL
brew services start mysql      # macOS
sudo service mysql start       # Linux
```

#### "Event handler failed"

```
Error: Event handler failed for event Transfer
```

**Solution**: Add error handling to your event handlers:

```typescript
indexer.onEvent({
  contractAddress: "0x...",
  abi: myAbi,
  eventName: "Transfer",
  handler: async (event, client, indexer) => {
    try {
      // Your event processing logic
      await processEvent(event);
    } catch (error) {
      console.error("Event processing failed:", error);
      // Implement your error handling strategy
    }
  },
});
```

#### "WebSocket connection lost"

```
WebSocket connection lost, attempting to reconnect...
```

**Solution**: This is usually normal behavior. AUCO has built-in reconnection logic, but you can monitor the connection:

```typescript
// Monitor indexer status
setInterval(() => {
  console.log("Indexer status:", indexer.getStatus());
}, 30000);
```

### Chain Reorganization Issues

#### "Reorg detected but handler failed"

```
Error: Reorg handler failed at block 123456
```

**Solution**: Implement robust reorg handling:

```typescript
indexer.onReorg({
  handler: async (forkedBlock) => {
    try {
      console.log(`Handling reorg at block ${forkedBlock.block_number}`);

      // Clear application cache
      await clearCacheFromBlock(forkedBlock.block_number);

      // Update external systems if needed
      await notifyExternalSystems(forkedBlock);
    } catch (error) {
      console.error("Reorg handling failed:", error);
      // Implement fallback strategy
    }
  },
});
```

## Best Practices for Error Prevention

### 1. Robust Event Handlers

Always implement error handling in event handlers:

```typescript
indexer.onEvent({
  contractAddress: "0x...",
  abi: myAbi,
  eventName: "Transfer",
  handler: async (event, client, indexer) => {
    try {
      // Process event
      await processTransferEvent(event);
    } catch (error) {
      // Log error with context
      console.error("Failed to process transfer event:", {
        error: error.message,
        event: event,
        blockNumber: event.block_number,
        transactionHash: event.transaction_hash,
      });

      // Optionally implement retry logic
      // await retryProcessing(event);
    }
  },
});
```

### 2. Connection Monitoring

Monitor database and node connections:

```typescript
// Monitor database health
setInterval(async () => {
  try {
    await client.query("SELECT 1");
    console.log("Database connection healthy");
  } catch (error) {
    console.error("Database connection issue:", error);
  }
}, 60000);
```

### 3. Graceful Shutdown

Always implement graceful shutdown:

```typescript
async function gracefulShutdown() {
  console.log("Starting graceful shutdown...");

  try {
    await indexer.stop();
    console.log("Indexer stopped successfully");
  } catch (error) {
    console.error("Error during shutdown:", error);
  }

  process.exit(0);
}

process.on("SIGINT", gracefulShutdown);
process.on("SIGTERM", gracefulShutdown);
```

## Getting Help

If you're still experiencing issues:

1. **Check the logs**: Enable debug logging to get more detailed information
2. **Search existing issues**: Check [GitHub Issues](https://github.com/Quantum3-Labs/auco/issues) for similar problems
3. **Create a new issue**: If you can't find a solution, create a [new issue](https://github.com/Quantum3-Labs/auco/issues/new) with:

   - Your configuration (without sensitive data)
   - Error messages and stack traces
   - Steps to reproduce the issue
   - Environment details (Node.js version, OS, database version)

4. **Join the community**: Participate in [GitHub Discussions](https://github.com/Quantum3-Labs/auco/discussions) for community support

## Debug Configuration

For maximum debugging information:

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  logLevel: LogLevel.DEBUG,
  rpcNodeUrl: "your-rpc-url",
  wsNodeUrl: "your-ws-url",
  database: {
    // your database config
  },
  // Add more detailed logging for development
  startingBlockNumber: "latest", // Start from latest to reduce initial load
});

// Enable additional debugging
process.env.DEBUG = "auco:*"; // Enable all debug namespaces
```
