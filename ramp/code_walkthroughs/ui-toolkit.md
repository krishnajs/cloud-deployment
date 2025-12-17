# UI Toolkit - Code Walkthrough

## Overview
The UI Toolkit is a TypeScript library that provides the core logic for interacting with Intel AMT features in the browser. It implements the RFB (Remote Frame Buffer) protocol for KVM and the Serial-over-LAN (SOL) protocol, running over WebSockets.

## Key Files & Components

### 1. KVM (Keyboard, Video, Mouse): `src/core/AMTDesktop.ts`
- **Class**: `AMTDesktop`
- **Responsibilities**:
  - Manages the canvas element to render the remote desktop.
  - Handles mouse and keyboard events and sends them to the remote device.
  - Uses `AMTKvmDataRedirector` to handle the binary data stream.

### 2. Data Redirection: `src/core/AMTKvmDataRedirector.ts`
- Handles the WebSocket connection to the MPS server (which proxies to the AMT device).
- Implements the protocol to unwrap/wrap AMT redirection messages.

### 3. Serial-over-LAN: `src/core/AMTTerminal.ts`
- **Class**: `AMTTerminal`
- **Responsibilities**:
  - Provides a terminal interface for the Serial-over-LAN feature.
  - Connects to the SOL endpoint on MPS.
  - Handles text encoding/decoding (xterm.js integration is common in consuming apps).

### 4. IDER (IDE Redirection): `src/core/AMTIDER.ts`
- Handles the redirection of local disk images (ISO/IMG) to the remote device as virtual boot media.

## Usage
This library is framework-agnostic (pure TypeScript) but is commonly consumed by:
- `ui-toolkit-react`: React wrappers around these core classes.
- `ui-toolkit-angular`: Angular wrappers.
- `sample-web-ui`: The reference implementation.

## Key Protocols
- **RFB**: Used for KVM. The toolkit implements a VNC-like client in the browser.
- **WebSocket**: The transport layer for all redirection traffic.
