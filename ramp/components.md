# Intel vPro & AMT Components - Open AMT Cloud Toolkit

This document provides a detailed breakdown of the components involved in the Intel vPro and Active Management Technology (AMT) ecosystem, specifically within the context of the Open AMT Cloud Toolkit.

## 1. Cloud Infrastructure Components
These components run on the server side (cloud or on-premise) to manage and provision the edge devices.

### Core Services
*   **MPS (Management Presence Server)**
    *   **Description:** A microservice that acts as the connectivity hub for managed devices.
    *   **Role:** Maintains persistent CIRA (Client Initiated Remote Access) tunnels with the AMT firmware on edge devices. It enables "out-of-band" management capabilities like KVM (Keyboard, Video, Mouse), Power Control, and Serial-over-LAN, even when the OS is down.
*   **RPS (Remote Provisioning Server)**
    *   **Description:** A microservice responsible for the activation and configuration of Intel AMT devices.
    *   **Role:** Manages provisioning profiles (defining *how* a device should be configured) and handles the initial activation request from the RPC. It generates the necessary configuration data (passwords, network settings) and stores secrets securely.
*   **MPS Router**
    *   **Description:** A routing service that works in conjunction with MPS.
    *   **Role:** Helps route CIRA connections and API requests to the appropriate MPS instance, facilitating scalability and connection management.
*   **Web UI (Sample Web UI / UI Toolkit)**
    *   **Description:** A web-based dashboard for IT administrators.
    *   **Role:** Provides a visual interface to manage devices, view status, create provisioning profiles, and execute management actions (power on/off, KVM session) via the MPS and RPS APIs.

### Infrastructure & Support Services
*   **Kong (API Gateway)**
    *   **Description:** An open-source API Gateway.
    *   **Role:** Acts as the single entry point for all external traffic. It handles SSL termination, request routing (directing traffic to RPS, MPS, or WebUI), and API security.
*   **PostgreSQL (Database)**
    *   **Description:** A relational database management system.
    *   **Role:** Stores persistent data for RPS and MPS, including device records, provisioning profiles, CIRA connection state, and audit logs.
*   **Vault (HashiCorp Vault)**
    *   **Description:** A secrets management tool.
    *   **Role:** Securely stores sensitive information such as AMT passwords, WiFi keys, and cryptographic certificates used for provisioning.
*   **Consul (Optional)**
    *   **Description:** A service networking solution.
    *   **Role:** Provides service discovery and dynamic configuration for the microservices (MPS, RPS).
*   **Mosquitto (Optional)**
    *   **Description:** An MQTT message broker.
    *   **Role:** Facilitates event-driven architecture, allowing components to publish and subscribe to events (e.g., "Device Connected", "Provisioning Failed").

---

## 2. Device Components (OS Level)
These components run within the Operating System (Windows or Linux) of the managed edge device.

*   **RPC (Remote Provisioning Client) / RPC-Go**
    *   **Description:** A command-line utility (available in C++ and Go) used to activate Intel AMT.
    *   **Role:**
        *   **Provisioning:** Connects to the RPS to retrieve a configuration profile.
        *   **Configuration:** Applies the configuration to the AMT firmware locally.
        *   **Lifecycle:** Typically runs once during setup (or for maintenance) and then exits. It is *not* a persistent background service.
*   **LMS (Local Manageability Service)**
    *   **Description:** A background service running on the OS.
    *   **Role:** Acts as a local proxy that listens on `localhost:16992`. It allows local applications (including RPC) to communicate with the AMT firmware using standard TCP/IP, translating those requests into driver calls.
*   **MEI Driver (Management Engine Interface)**
    *   **Description:** The kernel-level device driver (Windows: `Intel(R) Management Engine Interface`, Linux: `/dev/mei0`).
    *   **Role:** The bridge between the OS software (User Space) and the hardware firmware. It transmits commands over the HECI (Host Embedded Controller Interface) bus.

---

## 3. Device Components (Firmware Level)
These components run directly on the hardware, independent of the main Operating System.

*   **Intel AMT (Active Management Technology) Firmware**
    *   **Description:** The management logic embedded in the CSME.
    *   **Role:** Executes management commands (Power Control, KVM, etc.). It maintains the CIRA tunnel to the MPS server and enforces security policies. It remains active as long as the device has power and a network connection, even if the OS is crashed or missing.
*   **CSME (Converged Security and Management Engine)**
    *   **Description:** The isolated microcontroller and execution environment on the motherboard chipset.
    *   **Role:** The hardware "brain" that runs the AMT firmware. It is separate from the main CPU.
*   **Network Interface (NIC)**
    *   **Description:** The physical network hardware (Ethernet or WiFi).
    *   **Role:** Provides the physical link to the network. Intel AMT has direct access to the NIC (via the CSME), allowing it to send and receive traffic independently of the OS network stack.
