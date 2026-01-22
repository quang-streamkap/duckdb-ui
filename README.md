# DuckDB UI Extension

A [DuckDB extension](https://duckdb.org/docs/stable/core_extensions/ui.html) providing a browser-based user interface.

This repository contains both the extension, implemented in C++, and some packages used by the user interface, implemented in TypeScript.

While most of the user interface code is not yet publicly available, more of it will added here over time.

## Extension

The primary structure of this repository is based on the [DuckDB extension template](https://github.com/duckdb/extension-template).

To build the extension:

```sh
make
```

This will create the following binaries:

```sh
./build/release/duckdb                              # DuckDB shell with UI extension
./build/release/test/unittest                       # Test runner
./build/release/extension/ui/ui.duckdb_extension    # Loadable extension binary
```

- `duckdb` is the binary for the duckdb shell with the extension code automatically loaded.
- `unittest` is the test runner of duckdb. Again, the extension is already linked into the binary.
- `ui.duckdb_extension` is the loadable binary as it would be distributed.

To run the extension code, simply start the shell with `./build/release/duckdb`.

To start the UI from the command line:

```
./build/release/duckdb -ui
```

To start the UI from SQL:
```
call start_ui();
```

For more usage details, see the [documentation](https://duckdb.org/docs/stable/core_extensions/ui.html).

## Configuration

### Prerequisites

Before configuring extension settings, ensure the UI extension is loaded. The steps depend on how DuckDB is started:

**For loadable extensions (Docker, installed extensions):**

1. Start DuckDB with the `-unsigned` flag to enable unsigned extensions:
   ```bash
   duckdb -unsigned
   ```
   
   Or using Docker:
   ```bash
   docker run --rm -it -v "$(pwd):/workspace" -w /workspace --net host --entrypoint duckdb duckdb/duckdb -unsigned
   ```
   
   Or using Docker Compose (see `docker-compose.yml`):
   ```bash
   docker-compose run --rm duckdb
   ```

2. Once DuckDB is running, install and load the extension:
   ```sql
   INSTALL ui;
   LOAD ui;
   ```

   **Important:** The `-unsigned` flag already enables unsigned extensions at startup. You **do not need** (and cannot) set `allow_unsigned_extensions = true` at runtime. If you see an error about "Cannot change allow_unsigned_extensions setting while database is running", simply ignore it and proceed with installing/loading the extension - the flag has already enabled it.

   **Note:** If the `ui_local_host` setting is not available after loading the extension, the published extension version may not include this feature yet. In that case, you'll need to build the extension from source (see the "Extension" section above) and use a statically linked build, or wait for an updated extension release.

3. After loading the extension, the settings (like `ui_local_host`) will be available:
   ```sql
   SET ui_local_host = '0.0.0.0';
   ```

**For statically linked builds:**

If you're using a custom build where the extension is statically linked (like `./build/release/duckdb`), the extension and its settings are automatically available without needing to install/load it.

### Listening Interface

By default, the UI server listens on `0.0.0.0`, which allows connections from all network interfaces. You can configure the listening interface to restrict access:

```sql
SET ui_local_host = 'localhost';  -- Listen only on localhost (127.0.0.1)
```

Or bind to a specific IP address:

```sql
SET ui_local_host = '192.168.1.100';  -- Listen on specific IP
```

The default value is `'0.0.0.0'` to allow connections from any network interface. Use `'localhost'` to restrict access to local connections only, or specify a particular IP address to bind to a specific interface.

**Troubleshooting:** If you get an "unrecognized configuration parameter" error, verify the extension is loaded by checking if the `start_ui()` function is available:

```sql
SELECT start_ui();
```

If this fails, the extension is not loaded. For loadable extensions, ensure you've run `INSTALL ui;` and `LOAD ui;` first.

## User Interface Packages

Some packages used by the browser-based user interface can be found in the `ts` directory.

See the [README](ts/README.md) in that directory for details.

## Architectural Overview

The extension starts an HTTP server that both serves the UI assets (HTML, JavaScript, etc.)
and handles requests to run SQL and perform other DuckDB operations.

The server proxies requests for UI assets and fetches them from a remote server.
By default, this is `https://ui.duckdb.org`, but it can be [overridden](https://duckdb.org/docs/stable/core_extensions/ui.html#remote-url).

The server also exposes a number of HTTP endpoints for performing DuckDB operations.
These include running SQL, interrupting runs, tokenizing SQL text, and receiving events (such as catalog updates).
For details, see the `HttpServer::Run` method in [http_server.cpp](src/http_server.cpp).

The UI uses the TypeScript package [duckdb-ui-client](ts/pkgs/duckdb-ui-client/package.json) for communicating with the server.
See the [DuckDBUIClient](ts/pkgs/duckdb-ui-client/src/client/classes/DuckDBUIClient.ts) and [DuckDBUIClientConnection](ts/pkgs/duckdb-ui-client/src/client/classes/DuckDBUIClientConnection.ts) classes exposed by this package for details.
