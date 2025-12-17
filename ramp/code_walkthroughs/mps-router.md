# MPS Router - Code Walkthrough

## Overview
MPS Router is a Go-based high-performance proxy and load balancer. It sits in front of multiple MPS instances and routes incoming CIRA connections to the correct instance to ensure session stickiness.

## Key Files & Entry Points

### 1. Entry Point: `cmd/main.go`
- **Function**: `main()` -> `run()`
- **Responsibilities**:
  - Parses command-line flags and environment variables (`MPS_CONNECTION_STRING`, `PORT`).
  - Selects the database implementation (Postgres or Mongo) based on the connection string.
  - Starts the proxy server via `startServer`.

### 2. Proxy Logic: `internal/proxy/`
- Contains the core routing logic.
- Inspects the initial CIRA connection packet (CONNECT request) to identify the device GUID.
- Queries the database to see if this device is already connected to a specific MPS instance.
- Forwards the connection to the target MPS.

### 3. Database Abstraction: `internal/db/`
- Defines the `Manager` interface.
- Implementations for `PostgresManager` and `MongoManager`.
- Used to look up device-to-MPS mappings.

## Startup Flow
1.  **Init**: Parse flags (`-health`) and Env (`MPS_CONNECTION_STRING`).
2.  **DB Connect**: Initialize the DB client.
3.  **Health Check**: If `-health` flag is present, check DB connectivity and exit.
4.  **Serve**: Start listening on the configured port (default 8003) and proxy traffic to the MPS target (default `mps:3000`).

## Key Concepts
- **Stickiness**: Ensures that a device always reconnects to the same MPS instance if a session is active, or distributes load if new.
- **Transparent Proxy**: The router acts as a transparent TCP proxy after the initial routing decision.
