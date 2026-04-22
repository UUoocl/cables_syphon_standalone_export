# Cables Syphon Standalone Export

This project is a standalone export of a Cables.gl patch featuring a custom User Operator (`SyphonOut`) that publishes real-time texture data to a Syphon server. This allows for high-performance video streaming from a Cables application to other software like OBS, MadMapper, or Resolume on macOS.

## 🚀 Technology Stack

- **[Cables.gl](https://cables.gl/)**: A flexible, browser-based visual programming environment for creating interactive web graphics.
- **Electron**: The standalone runtime environment that allows Cables patches to run as desktop applications with access to Node.js.
- **Node.js**: Powers the backend logic and native module integration.
- **[node-syphon](https://github.com/benoitlahoz/node-syphon)**: A native Node.js module that provides bindings for the Syphon framework on macOS.

## 🛠 The SyphonOut Operator

The core component of this project is the **`Ops.Extension.Standalone.SyphonOut`** operator. 

### Implementation Details
- **Location**: `ops/Ops.Extension.Standalone.SyphonOut/`
- **Mechanism**: The operator captures the WebGL texture from the Cables render chain and publishes it to a Syphon server.
- **Performance**: Optimized for macOS, aiming for low-latency texture sharing. 

### Operator Inputs
- **Render (Trigger)**: Should be connected to the main render loop to trigger the publish on every frame.
- **Texture (Texture)**: The WebGL texture to be published.
- **Server Name (String)**: The name of the Syphon server as it will appear in client applications (Default: `Cables_Output`).

## 📥 Setup & Installation

To run this standalone patch, ensure the following requirements are met:

1. **OS**: macOS (Required for Syphon).
2. **Native Dependencies**: The project includes `node-syphon` as a native dependency.
3. **Electron Configuration**: Ensure your Electron environment is configured with `nodeIntegration: true` and `contextIsolation: false` to allow the operator to `require` the native module.

### Manual Dependency Install
If dependencies are missing, run the following in the operator directory:
```bash
cd ops/Ops.Extension.Standalone.SyphonOut
npm install
```

## 📖 Usage

1. Open the Cables patch (`syphonOut.cables`) in your standalone Cables environment.
2. Ensure the `SyphonOut` operator is connected at the end of your render pipeline.
3. Start your Syphon client (e.g., OBS with Syphon plugin).
4. Select the server named in the operator (default: `Cables_Output`).

## 📋 Specification Reference

This implementation follows the technical requirements outlined in the **Syphon Out Spec**.

- **Spec File**: [.agent/syphon_out_op.md](file:///Users/jonwood/Github_local_dev/syphon-out-user-op/.agent/syphon_out_op.md)
- **Target Platform**: Apple Silicon (M1/M2/M3/M4) optimized.
- **Goal**: Zero-copy GPU texture sharing via IOSurface.

---
*Created with **Antigravity** for the cables.gl standalone ecosystem.*
