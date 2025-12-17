# IDER (IDE Redirection)

## Overview
IDE Redirection (IDER) allows a remote administrator to mount a local disk image (ISO, IMG, or floppy) to the managed device. The device sees this as a physical USB drive or CD-ROM attached to it. This is primarily used for remote OS installation, booting into recovery tools, or BIOS updates.

## Workflow Steps
1.  **User** selects a bootable ISO file in the Web UI.
2.  **Web UI** (or the MPS server) acts as the IDER Server, hosting the file.
3.  **MPS** sends a command to AMT to enable IDER redirection.
4.  **AMT** connects to the IDER channel.
5.  **Device** (BIOS/OS) detects a new storage device (Virtual CD-ROM).
6.  **User** triggers a reboot (often using Boot Control to force boot from CD-ROM).
7.  **BIOS** attempts to boot from the virtual drive.
8.  **AMT** intercepts the disk read requests from the BIOS.
9.  **AMT** forwards the read requests to MPS/Web UI.
10. **MPS/Web UI** reads the requested sectors from the ISO file and sends them back to AMT.

## Workflow Diagram
```mermaid
flowchart TD
    A["ISO File"] -->|Read Sectors| B["Browser MPS"]
    B -->|IDER Protocol| C["Device AMT"]
    C -->|Emulate USB Drive| D["BIOS OS"]
    D -->|Read File| C
```

## Sequence Diagram
```mermaid
sequenceDiagram
    participant Browser
    participant MPS
    participant AMT
    participant BIOS

    Browser->>MPS: Select ISO & Start IDER
    MPS->>AMT: Enable IDER
    AMT-->>MPS: IDER Active
    MPS->>AMT: Reboot to CD-ROM
    AMT->>BIOS: Force Boot Option
    BIOS->>AMT: Read Sector 0
    AMT->>MPS: Request Sector 0
    MPS->>Browser: Fetch Data
    Browser-->>MPS: Data
    MPS-->>AMT: Sector Data
    AMT-->>BIOS: Sector Data
```
