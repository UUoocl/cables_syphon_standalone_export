# Antigravity Spec: `Ops.User.SyphonOut`

## 1. Project Context
* **Platform:** cables.gl Standalone (Electron-based).
* **Hardware:** Apple Silicon (M1 Pro / M4), 16GB Unified Memory.
* **Dependency:** `node-syphon` (by benoitlahoz).
* **Goal:** Zero-copy GPU texture sharing via Syphon.

## 2. Implementation Requirements

### A. Node.js Environment Setup
Antigravity must ensure the Electron environment allows native module access.
* **Requirement:** Enable `nodeIntegration: true` and `contextIsolation: false` in the Electron window settings (or via a preload script).
* **Dependency Install:** Run `npm install node-syphon` within the cables.gl standalone root directory.

### B. The Operator (Op) Logic
**File Path:** `src/ops/user/SyphonOut/SyphonOut.js`

**Antigravity Prompt Instruction:** *Generate a cables.gl Op that creates a Syphon server on initialization and publishes the input WebGL texture using the IOSurface sharing method on every render frame.*

* **Inputs:**
    * `inTrigger`: Trigger to execute the publish.
    * `inTexture`: Texture input from the cables.gl render chain.
    * `serverName`: String input to name the Syphon server (default: "Cables_Output").
* **Execution Logic:**
    1. **`onInit`**: Initialize the `SyphonOpenGLServer` from `node-syphon`.
    2. **`onRender`**: 
        * Extract the `glTexture` ID from the `inTexture` object.
        * Use the library's internal `IOSurface` binding if available, or fall back to the native `publish` call that supports macOS texture handles.
        * Ensure the width and height match the incoming texture dimensions.

---

## 3. Core Code Structure (Blueprint)

```javascript
const { SyphonOpenGLServer } = require('node-syphon');

/**
 * @name Ops.User.SyphonOut
 * @description Publishes a texture to Syphon via IOSurface for OBS.
 */
const inTrigger = op.inTrigger("Render");
const inTexture = op.inTexture("Texture");
const inName = op.inString("Server Name", "Cables_Output");

let server = null;
let currentName = "";

inTrigger.onTriggered = () => {
    const tex = inTexture.get();
    if (!tex || !tex.glTexture) return;

    // Lazy init or rename server
    if (!server || currentName !== inName.get()) {
        if (server) server.stop();
        currentName = inName.get();
        server = new SyphonOpenGLServer(currentName);
    }

    // High-performance publish
    // node-syphon handles the IOSurface sharing internally when passed the GL ID on Apple Silicon
    server.publish(tex.glTexture, tex.width, tex.height);
};

op.onDelete = () => {
    if (server) server.stop();
};
```

---

## 4. Critical Performance Tuning for Antigravity

* **Texture Format:** Force the Op to check if the incoming texture is `GL_TEXTURE_2D`. If the Electron context uses `GL_TEXTURE_RECTANGLE`, provide a toggle or auto-detection to prevent "black screen" issues in OBS.
* **Garbage Collection:** instruct the Agent to wrap the server instance in a persistent variable outside the `onTriggered` scope to prevent the Syphon server from being garbage collected during 4K high-load scenarios.
* **Syncing:** Ensure the `inTrigger` is connected to the **final** output of the cables.gl patch (e.g., after the `MainLoop` or a `Sequence` op) to avoid capturing incomplete frames.

## 5. Verification Checklist
1. **CPU Usage:** The `WindowServer` process should remain stable; high CPU indicates a fallback to pixel-copying.
2. **Alpha Transparency:** Verify that OBS "Syphon Client" sees the alpha channel (transparency) correctly from the cables.gl background.
3. **M4 NPU Isolation:** Ensure the Vision Framework tasks (segmentation) are still running on the NPU while this Op handles the GPU handoff.
