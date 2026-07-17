# WebRTC Low-Bandwidth Laptop Audio Streamer

Project - Backup - https://vdo.ninja./?push=kZuNmdH http://bit.ly/4prqRVT

A lightweight, single-page browser application that uses WebRTC to capture and stream system/laptop audio while aggressively optimizing for low-bandwidth environments.

Typical WebRTC audio configurations default to high-fidelity stereo streaming (~64kbps+). This project demonstrates how to hook into system media layers via browser APIs and modify the underlying **Session Description Protocol (SDP)** to force WebRTC's native **Opus** codec into a strict, highly efficient narrowband speech profile (~12kbps) with silence suppression.

---

## 🚀 Features

* **Native System Audio Capture**: Leverages `getDisplayMedia` to hook straight into desktop/tab audio channels without requiring external audio drivers or virtual cables.
* **Aggressive Video Dropping**: Instantly discards the accompanying video track to minimize CPU overhead and conserve visual pipeline bandwidth.
* **SDP Codec Injection**: Manually re-engineers WebRTC network handshakes to force low-bitrate configurations:
  * `maxaveragebitrate=12000` (Caps throughput to ~12kbps)
  * `stereo=0` (Downmixes to high-efficiency mono)
  * `usedtx=1` (Activates Discontinuous Transmission to completely halt network traffic during silence)
* **Single-File Zero Dependency Architecture**: Written entirely in vanilla HTML5, CSS3, and modern JavaScript.

---

## 🛠️ How It Works

WebRTC does not natively provide a discrete "Share Desktop Sound Only" API due to browser sandbox security constraints. Instead, we have to request a screen or tab capture, extract the accompanying hardware audio path, and discard the rest.


```

[System Audio Matrix] ---> getDisplayMedia() ---> [Separate Video / Audio]
|
(Kill Video / Isolate Audio Only)
|
[WebRTC Peer Tunnel] <--- Inject Low-Bandwidth SDP <--- [Audio Track]
(Mono / 12kbps / DTX)

```

---

## 💻 Quick Start

Because browser security models tightly restrict media APIs like `getDisplayMedia` to secure context layers, the file **cannot** simply be opened from your local filesystem (`file://`). It must be served over an HTTP or HTTPS loopback engine.

### Method 1: VS Code (Easiest)
1. Open the project folder in **Visual Studio Code**.
2. Install the **Live Server** extension.
3. Click **Go Live** in the bottom right corner of the window.

### Method 2: Node.js / Terminal
Navigate to the directory containing your `index.html` file and spin up an instant local web server:

```bash
# Using npx (built into Node)
npx serve

# Or using Python 3
python3 -m http.server 8000

```

Open your browser and navigate to `http://localhost:3000` (or `http://localhost:8000`).

---

## 📋 Step-by-Step Usage Instructions

1. Open the application interface in your browser.
2. Ensure the **"Force Low-Bandwidth Mode"** checkbox is toggled on.
3. Click the **"1. Capture & Stream Audio"** button.
4. When the browser prompt pops up, choose the window or browser tab you wish to share.
5. **CRITICAL STEP:** Look at the bottom-left corner of the browser sharing prompt and check the box that says **"Share system audio"** (or **"Share tab audio"**). If this is not checked, the browser will block the stream.
6. Click **Share**. The application will automatically isolate the audio channel and establish a peer loopback link.

> ⚠️ **Audio Feedback Warning:** The application's receiver playback component is muted by default. If you unmute it while capturing audio on the exact same laptop, your microphone/speakers will create a piercing, recursive audio loop feedback screech. Unmute with care or use headphones!

---

## 🔬 Deep Dive: The Codec Hack

WebRTC negotiates connections using text-based wrappers known as SDP. To modify the media behaviors dynamically, the application parses the SDP string before passing it down to the engine, hunting down the Opus payload configurations (`opus/48000`):

```javascript
// A look inside the SDP override engine:
lines[j] = `a=fmtp:${payloadType} maxaveragebitrate=12000;stereo=0;useinbandfec=1;usedtx=1`;

```

* **`maxaveragebitrate=12000`**: Drastically restricts data limits from heavy music-grade streams down to a highly responsive conversational frame size.
* **`usedtx=1`**: Tells the browser engine to stop blasting empty packets across the internet when there is no active noise or sound playing from your device.

---

## 🌐 Moving Past Loopback (True Production Setup)

To demonstrate the capture mechanics without infrastructural hurdles, this app runs both the sender and receiver connections inside the **same tab**.

To transform this into a real-world multi-device application:

1. Extract the `peerConnectionSender` logic into a dedicated `host.html` page.
2. Extract the `peerConnectionReceiver` logic into a `client.html` page.
3. Connect them using a minimal **Signaling Server** (such as a simple Node.js Node-WS or Socket.io server) to dynamically transmit the initial Offer and Answer SDP packets between different machines over the network.
