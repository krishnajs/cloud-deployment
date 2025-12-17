# MPS (Management Presence Server) - Code Walkthrough

## Overview
MPS is a Node.js/TypeScript microservice that acts as the CIRA (Client Initiated Remote Access) gateway. It maintains persistent connections with edge devices and proxies management traffic.

## Key Files & Entry Points

### 1. Entry Point: `src/index.ts`
- **Function**: `main()`
- **Responsibilities**:
  - Loads configuration and environment variables.
  - Connects to the Service Manager (Consul) if enabled.
  - Initializes the Database (`DbCreatorFactory`) and Secrets Manager (`SecretManagerCreatorFactory`).
  - Loads TLS certificates (`loadCertificates`).
  - Starts two main servers:
    - **MPSServer**: Listens for CIRA connections from devices (port 4433).
    - **WebServer**: Listens for API requests from the UI/RPS (port 3000).

### 2. CIRA Handling: `src/server/mpsserver.ts`
- Manages the raw TCP/TLS socket connections from Intel AMT devices.
- Handles the APF (AMT Port Forwarding) protocol handshake.
- Maintains a map of connected devices.

### 3. API Handling: `src/server/webserver.ts`
- Exposes REST endpoints for power actions, KVM, and redirection.
- Proxies traffic to the connected device via the `MPSServer` instance.

### 4. Inter-Service Communication
- **MQTT**: `src/utils/MqttProvider.ts` publishes device connection/disconnection events.
- **Database**: Shares the same PostgreSQL database with RPS for device inventory.

## Startup Flow
1.  **Config & Env**: Parse environment variables.
2.  **Dependencies**: Wait for DB and Vault to be healthy.
3.  **Certificates**: Load SSL certificates for TLS termination (if not handled by Kong/Router).
4.  **Listen**: Start both `MPSServer` (Device facing) and `WebServer` (User facing).

## Key Classes
- `MPSServer`: The core CIRA gateway logic.
- `WebServer`: The management API.
- `ConsulService`: Service discovery integration.
