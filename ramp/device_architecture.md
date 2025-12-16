# Device Architecture (Edge Node)

This diagram illustrates the internal architecture of a managed device, highlighting the separation between the Operating System (OS) and the Hardware/Firmware (CSME), and how local components interact with Intel AMT.

```mermaid
graph TD
    %% Styles
    classDef userSpace fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:black;
    classDef kernelSpace fill:#ffe0b2,stroke:#f57c00,stroke-width:2px,color:black;
    classDef hardware fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:black;
    classDef network fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px,color:black;

    subgraph "Operating System (Windows / Linux)"
        subgraph "User Space"
            RPC["RPC / RPC-Go<br>(Provisioning Tool)"]:::userSpace
            LMS["LMS Service<br>(Local Manageability Service)"]:::userSpace
            LocalApps["3rd Party Mgmt Apps<br>(Scripts, Agents)"]:::userSpace
        end

        subgraph "Kernel Space"
            MEI["MEI Driver<br>(/dev/mei0 or HECI)"]:::kernelSpace
            NetDriver["OS Network Driver"]:::kernelSpace
        end
    end

    subgraph "Hardware / Firmware (Independent of OS)"
        CSME["Intel CSME<br>(Converged Security & Mgmt Engine)"]:::hardware
        AMT["Intel AMT Firmware<br>(Running on CSME)"]:::hardware
        NIC["Network Interface Card<br>(PHY/MAC)"]:::hardware
    end

    subgraph "External Network"
        Cloud["MPS / RPS"]:::network
    end

    %% Local Communication Flow
    RPC -- "1. Provisioning Config" --> LMS
    RPC -. "Fallback (Direct Access)" .-> MEI
    
    LocalApps -- "http://localhost:16992" --> LMS
    LMS -- "Proxy Requests" --> MEI
    
    MEI -- "HECI Bus (Internal)" --> CSME

    %% Firmware Logic
    CSME --- AMT
    
    %% Network Flow - The Split
    NetDriver -- "OS Traffic" --> NIC
    AMT -- "OOB Traffic (CIRA)" --> NIC
    
    %% External Connection
    NIC -- "TCP/IP" --> Cloud

    %% Note on OOB
    linkStyle 7 stroke:#01579b,stroke-width:3px;
    linkStyle 8 stroke:#01579b,stroke-width:3px;
```

## Key Concepts

### 1. User Space vs. Hardware
*   **User Space:** Where standard applications (like RPC) and services (like LMS) run. They depend on the OS being active.
*   **Hardware (CSME):** Where Intel AMT runs. It is a separate execution environment on the motherboard. It operates independently of the main CPU and OS.

### 2. The Role of MEI
The **MEI (Management Engine Interface)** driver is the only bridge between the OS and the Firmware. Software cannot "talk" to the firmware without passing through this driver.

### 3. The Role of LMS
**LMS** acts as a translator. It allows applications to use standard web protocols (HTTP/S) to talk to the firmware, which expects low-level HECI bus commands. This simplifies development for local management tools.

### 4. Out-of-Band (OOB) Networking
Notice that **Intel AMT** has a direct line to the **NIC**. This allows it to send and receive network packets (like the CIRA tunnel to the cloud) even if the **OS Network Driver** is crashed or the OS is missing entirely.
