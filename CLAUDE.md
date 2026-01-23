# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DuckDB UI is a DuckDB extension that provides a browser-based user interface. It contains:
- C++ extension code (HTTP server, connection management, event handling)
- TypeScript packages for client-side communication and data handling

The extension starts an HTTP server that serves UI assets (proxied from `https://ui.duckdb.org` by default) and handles SQL execution requests.

## Build Commands

### C++ Extension
```bash
make                    # Build everything
make test               # Run SQLLogicTests
make test_debug         # Run tests in debug mode
```

Build outputs:
- `./build/release/duckdb` - DuckDB shell with UI extension
- `./build/release/test/unittest` - Test runner
- `./build/release/extension/ui/ui.duckdb_extension` - Loadable extension binary

### TypeScript (in `ts/` directory)
```bash
pnpm install            # Install dependencies and build src
pnpm build              # Build all packages (src and test)
pnpm build:src          # Build src only
pnpm build:test         # Build test only
pnpm build:watch        # Watch mode
```

## Testing

### C++ Tests
```bash
make test               # Run all tests
```
Tests are in `/test/sql/` using SQLLogicTest format.

### TypeScript Tests
```bash
cd ts
pnpm test               # Run all package tests
pnpm test:watch         # Watch mode

# Run tests for a specific package:
cd ts/pkgs/duckdb-ui-client
pnpm test
```
Tests use Vitest (Node or Browser mode with Chrome).

## Linting/Formatting

### TypeScript
```bash
cd ts
pnpm check              # Check formatting + linting
pnpm format:check       # Check formatting only
pnpm format:write       # Fix formatting (Prettier)
pnpm lint               # Check linting rules (ESLint)
```

## Architecture

### C++ Components (`src/`)
- **ui_extension.cpp** - Extension entry point; registers SQL functions (`start_ui()`, `stop_ui_server()`, `get_ui_url()`, etc.)
- **http_server.cpp** - HTTP server with endpoints: `/ddb/run` (execute SQL), `/ddb/interrupt`, `/ddb/tokenize`, `/localEvents` (SSE for catalog changes)
- **state.cpp** - Connection management via `UIStorageExtensionInfo`
- **event_dispatcher.cpp** - Server-sent events for UI clients
- **watcher.cpp** - Background thread monitoring catalog changes

### TypeScript Packages (`ts/pkgs/`)
- **@duckdb/ui-client** - Main client for HTTP server communication (`DuckDBUIClient`, `DuckDBUIClientConnection`)
- **@duckdb/data-values** - DuckDB value representations in TypeScript
- **@duckdb/data-types** - DuckDB type representations
- **@duckdb/data-reader** - Utilities for reading tabular data

### Data Flow
```
Browser → @duckdb/ui-client → HTTP Server (port 8080) → DuckDB Engine
                                    ↓
            ← Binary serialization ← Query Results
```

## Running the UI

```bash
./build/release/duckdb -ui          # From command line
# Or from SQL:
call start_ui();
```
