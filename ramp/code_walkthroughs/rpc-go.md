# RPC-Go (Remote Provisioning Client) - Code Walkthrough

## Overview
RPC-Go is a CLI utility written in Go. It runs on the edge device and facilitates the activation (provisioning) of Intel AMT by communicating with the local firmware and the remote RPS.

## Key Files & Entry Points

### 1. Entry Point: `cmd/rpc/main.go`
- **Function**: `main()`
- **Responsibilities**:
  - **Access Check**: Calls `checkAccess()` to verify that the Intel MEI driver is loaded and the process has root/admin privileges.
  - **Flag Parsing**: Uses `parseCommandLine` to handle arguments like `-u` (URL), `-p` (Profile), etc.
  - **Execution**: Calls `runRPC()`, which delegates to either `local.ExecuteCommand` or `rps.ExecuteCommand`.

### 2. AMT Communication: `internal/amt/`
- Wraps the interaction with the HECI driver.
- `NewAMTCommand()` initializes the interface to talk to the CSME firmware.

### 3. RPS Interaction: `internal/rps/`
- Handles the WebSocket connection to the RPS server.
- Implements the provisioning protocol (handshake, configuration application).

### 4. Local Configuration: `internal/local/`
- Handles "Local" activation mode (ACM) where configuration is applied directly without a remote server, often using a USB key or local file.

## Startup Flow
1.  **Pre-flight**: Check for MEI driver and permissions.
2.  **Parse Args**: Determine if running in `--local` mode or connecting to a server (`-u`).
3.  **Connect**:
    - If Remote: Establish WebSocket connection to RPS.
    - If Local: Read local configuration.
4.  **Provision**: Send AMT commands via HECI to apply the configuration (set password, activate network, etc.).

## Key Packages
- `pkg/heci`: Low-level interface to the Management Engine Interface driver.
- `pkg/wsman`: Constructs WS-Management messages for AMT control.
