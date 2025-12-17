# RPC (C++) - Code Walkthrough

## Overview
RPC (C++) is the legacy or alternative implementation of the Remote Provisioning Client. Like the Go version, it runs on the edge device to facilitate AMT activation. It uses `cpprestsdk` for networking.

## Key Files & Entry Points

### 1. Entry Point: `main.cpp`
- **Function**: `main()`
- **Responsibilities**:
  - **Privilege Check**: `cmd_is_admin()` ensures root/admin access.
  - **Arg Parsing**: `args_get_help`, `args_get_version`, etc.
  - **Command Dispatch**: Based on arguments, it calls functions like `activation_amt` or `info_get`.
  - **Timeout Thread**: Spawns a thread `timeout_thread_function` to kill the process if it hangs.

### 2. Activation Logic: `activation.cpp`
- Handles the core provisioning flow.
- Connects to the RPS server via WebSocket (`cpprest/ws_client.h`).
- Exchanges messages to configure AMT.

### 3. LMS Interaction: `lms.cpp`
- Interacts with the Local Manageability Service (LMS) to send WSMAN commands to the firmware.
- If LMS is not present, it may attempt direct HECI communication (though MicroLMS is often bundled).

### 4. Commands: `commands.cpp`
- Implements specific AMT commands (e.g., getting UUID, setting DNS).

## Startup Flow
1.  **Init**: Check admin privileges.
2.  **Parse**: Read command line args (`--url`, `--profile`, etc.).
3.  **Connect**: Establish WebSocket connection to the provisioning server.
4.  **Loop**: Process messages from the server and execute corresponding AMT commands via LMS/HECI.

## Key Libraries
- `cpprestsdk` (Casablanca): For WebSocket and JSON handling.
- `OpenSSL`: For secure connections.
