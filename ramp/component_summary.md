# Component Implementation & Deployment Summary

This document provides a technical summary of the key components in the Open AMT and EIM integration, detailing their implementation languages, code structure, and deployment models.

| Component | Implementation Type | Code Walkthrough Summary | Deployment Model |
| :--- | :--- | :--- | :--- |
| **RPS**<br>(Remote Provisioning Server) | **Node.js / TypeScript** | Built using a microservices architecture. Handles REST API requests for provisioning profiles, domains, and CIRA configurations. Interacts with PostgreSQL for data persistence and Vault for secrets. | **Docker Container**<br>(`intel/oact-rps`)<br>Deployed via Docker Compose or Kubernetes. |
| **MPS**<br>(Management Presence Server) | **Node.js / TypeScript** | Handles persistent TCP/TLS CIRA connections from edge devices. Implements the AMT redirection protocol (APF). Exposes a REST API for management actions (Power, KVM) and proxies traffic to the device. | **Docker Container**<br>(`intel/oact-mps`)<br>Deployed via Docker Compose or Kubernetes. |
| **MPS Router** | **Go** | A high-performance routing service. It inspects incoming CIRA connection requests and routes them to the appropriate MPS instance to ensure session stickiness and load balancing. | **Docker Container**<br>(`intel/oact-mpsrouter`)<br>Deployed as a gateway/load balancer. |
| **DM Manager**<br>(Device Management Mgr) | **Go** | Acts as an adapter between EIM and Open AMT. Implements gRPC/REST interfaces for the EIM Inventory. Contains logic to translate abstract EIM commands into specific MPS/RPS API calls. | **Binary / Docker**<br>Deployed as a service within the EIM cloud infrastructure. |
| **Web UI**<br>(Sample Web UI) | **React / Angular**<br>(TypeScript) | Single Page Application (SPA). Consumes MPS and RPS APIs via the Kong Gateway. Includes UI components for device lists, profile forms, and a KVM viewer (using WebSockets). | **Docker Container**<br>(Nginx)<br>Serves static assets via Nginx. |
| **RPC-Go**<br>(Remote Provisioning Client) | **Go** | CLI utility. Uses `pkg/heci` to communicate with the MEI driver via system calls. Uses `pkg/wsman` to construct AMT management messages. Handles the provisioning handshake with RPS. | **CLI Binary**<br>Executed on-demand by the PMA or manually. |
| **PMA**<br>(Platform Manageability Agent) | **Go** | System daemon. Reads EIM configuration (YAML). Automates the execution of the RPC binary. Monitors the health of the local AMT stack (LMS/MEI) and reports status to DM Manager. | **System Service**<br>(Systemd)<br>Installed via Debian/RPM package on the edge node. |
| **LMS**<br>(Local Manageability Service) | **C++** (MicroLMS)<br>or **C#** (Intel LMS) | Background service. Listens on `localhost:16992`. Acts as a proxy, accepting HTTP WSMAN requests and forwarding them to the firmware via the HECI driver. | **System Service**<br>Runs as a background daemon on the OS. |
| **Kong** | **Lua / Nginx** | API Gateway. Configured via `kong.yaml` (declarative). Handles SSL termination, route forwarding to MPS/RPS/WebUI, and authentication. | **Docker Container**<br>(`kong`)<br>Entry point for all ingress traffic. |

## External Dependencies

The following external services and infrastructure components are required for the full deployment and operation of the Open AMT Cloud Toolkit.

| Dependency | Category | Usage Description |
| :--- | :--- | :--- |
| **PostgreSQL** | **Database** | Primary data store for RPS and MPS. Stores device inventory, provisioning profiles, CIRA configs, and user accounts. Shared by multiple services. |
| **HashiCorp Vault** | **Security** | Secrets management engine. Securely stores sensitive data such as AMT passwords, WiFi profiles, and 802.1x certificates. RPS reads/writes secrets here. |
| **Mosquitto** | **Messaging** | MQTT Broker. Used for publishing device events and status updates. Allows external applications to subscribe to real-time alerts from MPS. |
| **Intel MEI Driver** | **Kernel Driver** | (Management Engine Interface) Linux kernel driver required on the edge device. Allows the OS (LMS/RPC) to communicate with the CSME firmware. |
| **Intel CSME Firmware** | **Hardware** | (Converged Security and Management Engine) The actual firmware running on the Intel vPro chipset. It hosts the AMT logic and network stack. |
