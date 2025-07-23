---
sidebar_position: 5
---

# Development

Information for developers who want to contribute to AUCO or work with the source code.

## Development Setup

### Prerequisites

1. **Node.js**: Version 18 or higher
2. **Git**: For version control
3. **Database**: PostgreSQL, SQLite, or MySQL for testing

### Getting Started

1. **Fork and clone** the repository:

   ```bash
   git clone https://github.com/YOUR_USERNAME/auco.git
   cd auco
   ```

2. **Install dependencies**:

   ```bash
   npm install
   ```

3. **Setup development environment**:

   ```bash
   # Start local PostgreSQL (if using Docker)
   docker run --name postgres-dev \
     -e POSTGRES_PASSWORD=postgres \
     -p 5432:5432 \
     -d postgres:13

   # Start local Starknet devnet
   npm run chain
   ```

4. **Run tests**:
   ```bash
   npm test
   ```

## Building from Source

### Build the Project

```bash
npm run build
```

This compiles TypeScript to JavaScript in the `dist/` directory.

### Watch Mode (Development)

```bash
npm run dev
```

Runs the compiler in watch mode for active development.

## Testing

### Running Tests

```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

### Test Structure

Tests are organized by component:

- `tests/unit/` - Unit tests for individual components
- `tests/integration/` - Integration tests with real databases
- `tests/e2e/` - End-to-end tests with real Starknet nodes

## Code Quality

AUCO maintains high code quality standards through automated tools.

### Linting

```bash
# Check code style
npm run lint

# Fix code style issues automatically
npm run lint:fix
```

### Formatting

```bash
# Format code with Prettier
npm run format

# Check if code is properly formatted
npm run format:check
```

### Type Checking

```bash
# Run TypeScript type checking
npm run typecheck
```

## Contributing

We welcome contributions! Please follow these guidelines:

### Code Style

- Use TypeScript for all new code
- Follow the existing code style (enforced by ESLint and Prettier)
- Write tests for new functionality
- Update documentation as needed

### Pull Request Process

1. **Create a feature branch**:

   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** following the code style guidelines

3. **Run tests and quality checks**:

   ```bash
   npm test
   npm run lint
   npm run typecheck
   ```

4. **Commit your changes** with a descriptive message:

   ```bash
   git commit -m "feat: add support for new feature"
   ```

5. **Push your branch** and create a pull request

### Commit Message Format

Follow conventional commits format:

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `style:` - Code style changes
- `refactor:` - Code refactoring
- `test:` - Test-related changes
- `chore:` - Build process or auxiliary tool changes

## Project Structure

```
auco/
├── src/                    # Source code
│   ├── core/              # Core indexer logic
│   ├── database/          # Database adapters
│   ├── types/             # TypeScript type definitions
│   └── utils/             # Utility functions
├── tests/                 # Test files
├── example/               # Example implementations
├── docs/                  # Documentation
└── scripts/               # Build and utility scripts
```

## Debugging

### Enable Debug Logging

```typescript
import { StarknetIndexer, LogLevel } from "auco";

const indexer = new StarknetIndexer({
  logLevel: LogLevel.DEBUG,
  // ... other config
});
```

### Environment Variables

Set debug environment variables:

```bash
# Enable all debug output
DEBUG=auco:*

# Enable specific debug namespaces
DEBUG=auco:database,auco:websocket
```

## Release Process

### Versioning

AUCO follows [Semantic Versioning](https://semver.org/):

- **Major**: Breaking changes
- **Minor**: New features (backward compatible)
- **Patch**: Bug fixes (backward compatible)

### Creating a Release

1. **Update version** in `package.json`
2. **Update CHANGELOG.md** with release notes
3. **Run tests** to ensure everything works
4. **Create a git tag**:
   ```bash
   git tag v1.2.3
   git push origin v1.2.3
   ```
5. **Publish to npm**:
   ```bash
   npm publish
   ```

## Performance Testing

### Benchmarking

Run performance benchmarks:

```bash
npm run benchmark
```

### Memory Profiling

Profile memory usage during development:

```bash
# Run with memory profiling
node --inspect --max-old-space-size=4096 dist/example/index.js
```

## Documentation

### Building Documentation

The documentation is built with Docusaurus:

```bash
# Install documentation dependencies
cd docs
npm install

# Start development server
npm run start

# Build documentation
npm run build
```

### Writing Documentation

- Use clear, concise language
- Include code examples for all features
- Add troubleshooting information for common issues
- Keep documentation up-to-date with code changes

## Getting Help

### Community Resources

- **GitHub Issues**: Report bugs or request features
- **GitHub Discussions**: Ask questions and share ideas
- **Discord**: Join our community chat (link in README)

### Maintainer Contact

For questions about contributing or the development process, reach out to the maintainers through GitHub issues or discussions.

## Acknowledgments

AUCO is built with excellent open-source libraries:

- [Starknet.js](https://github.com/0xs34n/starknet.js) - Starknet JavaScript library
- [abi-wan-kanabi](https://github.com/keep-starknet-strange/abi-wan-kanabi) - Type-safe ABI parsing
- [node-postgres](https://github.com/brianc/node-postgres) - PostgreSQL client
- [better-sqlite3](https://github.com/WiseLibs/better-sqlite3) - SQLite client
- [mysql2](https://github.com/sidorares/node-mysql2) - MySQL client

Thank you to all contributors and the broader Starknet ecosystem!
