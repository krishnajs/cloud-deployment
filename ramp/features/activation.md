# Activation Modes (ACM & CCM)

## Overview
Activation is the process of configuring the Intel AMT firmware on a device to allow remote management. The Open AMT Cloud Toolkit supports two primary modes: **Client Control Mode (CCM)** and **Admin Control Mode (ACM)**.

## 1. Client Control Mode (CCM)
CCM is the default mode when no trusted certificate is present on the device or when activated via host-based provisioning without a certificate match. It requires User Consent (a 6-digit code on screen) for sensitive operations like KVM.

### Workflow Steps
1.  **Administrator** creates a CCM Profile in RPS.
2.  **Administrator** generates a CIRA Config (optional, but standard for cloud).
3.  **Device** runs the RPC (Remote Provisioning Client) tool.
4.  **RPC** connects to RPS and requests activation.
5.  **RPS** sends the configuration (password, network, CIRA).
6.  **RPC** applies the configuration to the local firmware via LMS/HECI.
7.  **Firmware** is activated in CCM.

### Workflow Diagram
```mermaid
flowchart TD
    A["Admin"] -->|Create Profile| B["RPS"]
    C["Device RPC"] -->|Connect| B
    B -->|Send Config| C
    C -->|Apply Config| D["AMT Firmware"]
    D -->|Activate| E["CCM Mode"]
```

### Sequence Diagram
```mermaid
sequenceDiagram
    participant Admin
    participant RPC as Device (RPC)
    participant RPS
    participant AMT as AMT Firmware

    Admin->>RPS: Create CCM Profile
    RPC->>RPS: WebSocket Connect (Activate)
    RPS->>RPC: Challenge (Digest/Password)
    RPC->>RPS: Response
    RPS->>RPC: Send Configuration (Profile)
    RPC->>AMT: Apply Settings (HECI/LMS)
    AMT-->>RPC: Success
    RPC-->>RPS: Activation Complete
```

---

## 2. Admin Control Mode (ACM)
ACM provides a higher level of trust. It requires a certificate hash in the AMT firmware that matches a certificate held by RPS. It allows KVM and other features without requiring User Consent.

### Workflow Steps
1.  **Administrator** uploads a Provisioning Certificate (PFX) to RPS (Vault).
2.  **Administrator** creates an ACM Profile in RPS.
3.  **Device** runs RPC.
4.  **RPC** connects to RPS.
5.  **RPS** detects the device's certificate hashes (in the Hello message).
6.  **RPS** signs the configuration using the matching private key.
7.  **RPC** injects the signed configuration into the firmware.
8.  **Firmware** verifies the signature and activates in ACM.

### Workflow Diagram
```mermaid
flowchart TD
    A["Admin"] -->|Upload Cert| B["RPS"]
    C["Device RPC"] -->|Get Cert Hashes| D["AMT Firmware"]
    C -->|Send Hashes| B
    B -->|Sign Config| C
    C -->|Inject Signed Config| D
    D -->|Verify & Activate| E["ACM Mode"]
```

### Sequence Diagram
```mermaid
sequenceDiagram
    participant RPC as Device (RPC)
    participant RPS
    participant Vault
    participant AMT as AMT Firmware

    RPC->>AMT: Get Certificate Hashes
    RPC->>RPS: Connect (Send Hashes)
    RPS->>Vault: Retrieve Private Key
    RPS->>RPS: Sign Configuration Blob
    RPS->>RPC: Send Signed Config
    RPC->>AMT: Apply Signed Config
    AMT->>AMT: Verify Signature
    AMT-->>RPC: Success (ACM)
    RPC-->>RPS: Activation Complete
```
