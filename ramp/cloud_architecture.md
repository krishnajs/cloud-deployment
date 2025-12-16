# Cloud Infrastructure Architecture

This diagram details the server-side components of the Open AMT Cloud Toolkit, showing how microservices, databases, and security components interact to provide management capabilities.

```mermaid
graph TD
    %% Styles
    classDef gateway fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,color:black;
    classDef service fill:#e1f5fe,stroke:#0277bd,stroke-width:2px,color:black;
    classDef db fill:#e0f2f1,stroke:#00695c,stroke-width:2px,color:black;
    classDef external fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:black;

    subgraph "External World"
        Admin(("IT Admin<br>(Browser)")):::external
        Device(("Managed Device<br>(CIRA)")):::external
        RPC(("RPC Client")):::external
    end

    subgraph "API Gateway Layer"
        Kong["Kong API Gateway<br>(SSL Termination, Routing)"]:::gateway
    end

    subgraph "Microservices Layer"
        WebUI["Web UI Service<br>(Frontend Assets)"]:::service
        RPS["RPS<br>(Remote Provisioning Server)"]:::service
        MPS["MPS<br>(Management Presence Server)"]:::service
        Router["MPS Router<br>(Load Balancing CIRA)"]:::service
    end

    subgraph "Data & Support Layer"
        Postgres[("PostgreSQL<br>(Persistent Data)")]:::db
        Vault[("HashiCorp Vault<br>(Secrets Management)")]:::db
        Consul[("Consul<br>(Service Discovery)")]:::db
        Mosquitto[("Mosquitto MQTT<br>(Event Bus)")]:::db
    end

    %% Ingress Traffic
    Admin -- "HTTPS (443)" --> Kong
    RPC -- "HTTPS (443)" --> Kong
    Device -- "CIRA (4433)" --> Router

    %% Kong Routing
    Kong -- "/ui" --> WebUI
    Kong -- "/rps" --> RPS
    Kong -- "/mps" --> MPS

    %% Service Interactions
    RPS -- "Read/Write Profiles" --> Postgres
    RPS -- "Store/Retrieve Secrets" --> Vault
    RPS -- "Register/Discover" --> Consul
    RPS -- "Publish Events" --> Mosquitto

    MPS -- "Read/Write Device State" --> Postgres
    MPS -- "Retrieve Secrets" --> Vault
    MPS -- "Register/Discover" --> Consul
    MPS -- "Publish Events" --> Mosquitto

    %% Router Logic
    Router -- "Route Connection" --> MPS
    
    %% Database Logical Separation
    Postgres -. "rpsdb" .- RPS
    Postgres -. "mpsdb" .- MPS
```
