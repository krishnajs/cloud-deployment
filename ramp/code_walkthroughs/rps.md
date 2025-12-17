# RPS (Remote Provisioning Server) - Code Walkthrough

## Overview
RPS is a Node.js/TypeScript microservice responsible for handling the provisioning of Intel vPro devices. It manages profiles, domains, and CIRA configurations.

## Key Files & Entry Points

### 1. Entry Point: `src/Index.ts`
- **Function**: `startItUp()`
- **Responsibilities**:
  - Initializes `Configurator` to load environment variables.
  - Sets up `WebSocketListener` for handling incoming connections (though primarily MPS handles CIRA, RPS uses websockets for the Enterprise Assistant).
  - Connects to the MQTT broker via `MqttProvider`.
  - Initializes the Database connection via `DbCreatorFactory`.
  - Loads custom middleware.
  - Starts the Express app and listens on the configured port.

### 2. API Routes: `src/routes/index.ts`
- Defines the REST API endpoints (e.g., `/profiles`, `/domains`, `/admin`).
- Maps routes to controllers/handlers.

### 3. Data Access: `src/data/`
- Contains the repository pattern implementation for interacting with PostgreSQL.
- `DbCreatorFactory.ts` abstracts the DB creation logic.

### 4. Secrets Management: `src/secrets/`
- Handles interaction with HashiCorp Vault (or other secret providers) to store sensitive data like passwords and certificates.

## Startup Flow
1.  **Config Load**: `rc` module loads configuration.
2.  **DB Wait**: `waitForDB()` ensures Postgres is ready.
3.  **Secrets Wait**: `waitForSecretsManager()` ensures Vault is ready.
4.  **Server Start**: Express server starts, exposing API endpoints.

## Key Classes
- `Configurator`: Central configuration management.
- `MqttProvider`: Publishes events to the message bus.
- `WebSocketListener`: Handles WebSocket connections.
