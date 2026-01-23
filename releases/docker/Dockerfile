# Build stage
FROM ubuntu:22.04 AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    ninja-build \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy source code (submodules must be initialized locally first)
WORKDIR /src
COPY . .

# Build DuckDB with UI extension
RUN make -j$(nproc)

# Runtime stage
FROM ubuntu:22.04

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libssl3 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Copy built binary
COPY --from=builder /src/build/release/duckdb /usr/local/bin/duckdb

# Create data directory
RUN mkdir -p /data
WORKDIR /data

# Expose UI port
EXPOSE 4213

# Start DuckDB with UI
CMD ["duckdb", "-cmd", "LOAD 'ui'; CALL start_ui();"]
