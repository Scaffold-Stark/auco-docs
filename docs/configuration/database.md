---
sidebar_position: 1
---

# Database Configuration

AUCO supports multiple database backends for storing event and block data. Choose the database that best fits your needs.

## Supported Databases

- **PostgreSQL**: Recommended for production environments
- **SQLite**: Perfect for development and smaller applications
- **MySQL**: Alternative SQL database option

## PostgreSQL

PostgreSQL is the recommended database for production environments due to its robustness and advanced features.

### Basic Configuration

```typescript
import { StarknetIndexer } from "auco";

const indexer = new StarknetIndexer({
  // ... other config
  database: {
    type: "postgres",
    config: {
      connectionString:
        "postgresql://username:password@localhost:5432/database_name",
    },
  },
});
```

### Advanced Configuration

<details>
<summary>Individual Connection Options</summary>

```typescript
const indexer = new StarknetIndexer({
  // ... other config
  database: {
    type: "postgres",
    config: {
      host: "localhost",
      port: 5432,
      database: "starknet_indexer",
      user: "your_username",
      password: "your_password",
      ssl: false, // Set to true for production
      max: 20, // Maximum number of clients in the pool
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 2000,
    },
  },
});
```

</details>

### Setup Instructions

1. **Install PostgreSQL** (if not already installed):

   ```bash
   # macOS with Homebrew
   brew install postgresql

   # Ubuntu/Debian
   sudo apt-get install postgresql postgresql-contrib

   # Docker
   docker run --name postgres-auco -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres:13
   ```

2. **Create Database**:
   ```sql
   CREATE DATABASE starknet_indexer;
   CREATE USER auco_user WITH PASSWORD 'your_password';
   GRANT ALL PRIVILEGES ON DATABASE starknet_indexer TO auco_user;
   ```

:::tip Production Setup
For production environments, consider using connection pooling, SSL connections, and proper user permissions.
:::

## SQLite

SQLite is ideal for development, testing, and smaller applications that don't require a separate database server.

### Configuration

```typescript
import Database from "better-sqlite3";
import { StarknetIndexer } from "auco";

const indexer = new StarknetIndexer({
  // ... other config
  database: {
    type: "sqlite",
    config: {
      dbInstance: new Database("./starknet_indexer.db"),
    },
  },
});
```

### Advanced SQLite Options

<details>
<summary>SQLite Configuration Options</summary>

```typescript
import Database from "better-sqlite3";

const db = new Database("./starknet_indexer.db", {
  verbose: console.log, // Log SQL statements (development only)
  fileMustExist: false, // Create file if it doesn't exist
  timeout: 5000, // Busy timeout in milliseconds
  readonly: false, // Open in read-only mode
});

// Additional pragmas for performance
db.pragma("journal_mode = WAL"); // Write-Ahead Logging
db.pragma("cache_size = 10000"); // Cache size
db.pragma("temp_store = memory"); // Use memory for temporary storage

const indexer = new StarknetIndexer({
  // ... other config
  database: {
    type: "sqlite",
    config: {
      dbInstance: db,
    },
  },
});
```

</details>

### Setup Instructions

1. **Install better-sqlite3**:

   ```bash
   npm install better-sqlite3
   npm install --save-dev @types/better-sqlite3  # For TypeScript
   ```

2. **File Location**: The database file will be created automatically at the specified path.

:::info Memory Database
For testing, you can use an in-memory database by passing `:memory:` as the filename:

```typescript
new Database(":memory:");
```

:::

## MySQL

MySQL is a popular alternative to PostgreSQL with good performance characteristics.

### Basic Configuration

```typescript
import { StarknetIndexer } from "auco";

const indexer = new StarknetIndexer({
  // ... other config
  database: {
    type: "mysql",
    config: {
      connectionString:
        "mysql://username:password@localhost:3306/database_name",
    },
  },
});
```

### Advanced Configuration

<details>
<summary>Individual Connection Options</summary>

```typescript
const indexer = new StarknetIndexer({
  // ... other config
  database: {
    type: "mysql",
    config: {
      host: "localhost",
      port: 3306,
      database: "starknet_indexer",
      user: "your_username",
      password: "your_password",
      ssl: false,
      connectionLimit: 10,
      acquireTimeout: 60000,
      timeout: 60000,
      reconnect: true,
    },
  },
});
```

</details>

### Setup Instructions

1. **Install MySQL** (if not already installed):

   ```bash
   # macOS with Homebrew
   brew install mysql

   # Ubuntu/Debian
   sudo apt-get install mysql-server

   # Docker
   docker run --name mysql-auco -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 -d mysql:8.0
   ```

2. **Create Database**:
   ```sql
   CREATE DATABASE starknet_indexer;
   CREATE USER 'auco_user'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON starknet_indexer.* TO 'auco_user'@'localhost';
   FLUSH PRIVILEGES;
   ```

## Database Schema

AUCO automatically creates the necessary tables when it starts. The main tables include:

- **`blocks`**: Stores block information
- **`events`**: Stores parsed event data
- **`contracts`**: Tracks monitored contracts

:::warning Automatic Migrations
AUCO handles database schema creation and migrations automatically. Do not modify the schema manually unless you know what you're doing.
:::

## Performance Considerations

### PostgreSQL

- Use connection pooling for high-throughput applications
- Consider partitioning large tables by block number
- Set up proper indexes for your query patterns
- Use WAL mode for better concurrent access

### SQLite

- Enable WAL mode: `PRAGMA journal_mode = WAL;`
- Increase cache size: `PRAGMA cache_size = 10000;`
- Use memory for temporary storage: `PRAGMA temp_store = memory;`
- Consider file system performance (SSD recommended)

### MySQL

- Use InnoDB storage engine (default)
- Configure appropriate buffer pool size
- Set up proper indexes for your query patterns
- Consider read replicas for read-heavy workloads

## Connection String Examples

### PostgreSQL

```bash
# Basic
postgresql://user:pass@localhost:5432/dbname

# With SSL
postgresql://user:pass@localhost:5432/dbname?sslmode=require

# With connection pool settings
postgresql://user:pass@localhost:5432/dbname?max=20&idleTimeoutMillis=30000
```

### MySQL

```bash
# Basic
mysql://user:pass@localhost:3306/dbname

# With SSL and timezone
mysql://user:pass@localhost:3306/dbname?ssl=true&timezone=utc

# With connection limits
mysql://user:pass@localhost:3306/dbname?connectionLimit=10&acquireTimeout=60000
```

## Troubleshooting

### Common Issues

| Issue                 | Solution                                                       |
| --------------------- | -------------------------------------------------------------- |
| Connection refused    | Ensure database server is running and accessible               |
| Authentication failed | Check username, password, and permissions                      |
| Database not found    | Create the database first or check the name                    |
| SSL errors            | Configure SSL settings properly or disable SSL for development |
| Performance issues    | Check indexes, connection pooling, and database configuration  |

### Debugging Connection Issues

Enable debug logging to troubleshoot connection problems:

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  // ... other config
  logLevel: LogLevel.DEBUG, // Enable debug logging
  database: {
    // ... database config
  },
});
```

:::tip Getting Help
If you encounter database issues, check the [GitHub Issues](https://github.com/Quantum3-Labs/auco/issues) or join the [GitHub Discussions](https://github.com/Quantum3-Labs/auco/discussions) for community support.
:::
