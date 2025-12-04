# Pause for Permission Prompt in Chromium

Chromium implements its “pause for permission” behavior for **fetch()** and **XHR** using the `URLLoaderThrottle` system inside the Network Service. When a request targets a local network address, the browser inserts a throttle that freezes the request, triggers the Local Network Access (LNA) permission UI, and only resumes once the user chooses *Allow* or *Block*.

Here’s a clear breakdown of how it works — and why it behaves differently from WebSockets.

---

## 1. How the Pause Mechanism Works

### URLLoaderThrottle: The Checkpoint System

All resource loads (fetch, XHR, images, scripts) run through `URLLoader`.  
`URLLoader` supports a chain of throttles — C++ classes that can intercept request stages:

- `WillStartRequest`  
- `WillRedirectRequest`  
- `WillProcessResponse`  

Each throttle can return **DEFER**, which halts the request.

**Normal Flow:**  
Request → Throttle (Proceed) → Network

**LNA Flow:**  
Request → Throttle (Defer) → Paused → UI Prompt

---

## 2. Step-by-Step: What Happens During Pause

Example:

```js
fetch("http://192.168.1.5/data");
```
Step 1: Start
The Renderer process asks the Network Service to start the request. DNS resolves.

Step 2: Detection
LocalNetworkAccessCheck sees:

Target IP is private (192.168.x.x)

Initiator is a public website

Step 3: Deferral (the actual pause)
The throttle sets a defer flag (e.g., via PauseReadingBodyFromNet()).

Effect:
* Request stops progressing
* Connection is held in memory
* JavaScript fetch() promise remains pending

Step 4: IPC to Browser Process
The throttle sends a Mojo IPC message to the PermissionController.

Step 5: Show UI
The Browser Process displays the LNA permission prompt.

Step 6: Resume or Cancel
Allow: Browser sends IPC back → throttle calls Resume() → request continues.

Block: Throttle cancels with BLOCKED_BY_CLIENT → fetch rejects.



Why WebSockets Behave Differently :
WebSockets do not use URLLoader for their handshake. Instead, they rely on the CreateWebSocket interface, which establishes a separate bidirectional pipe.

