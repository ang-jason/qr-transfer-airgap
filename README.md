# Airgap QR Transfer
[![Netlify Status](https://api.netlify.com/api/v1/badges/afc5d566-e512-4952-8fc6-acc6a5555d3a/deploy-status)](https://app.netlify.com/projects/qr-air-send/deploys)<br>
**Move files between two devices using only a screen and a camera. No internet. No cables. No Bluetooth.**

Designed for air-gapped environments where network access is restricted or unavailable — isolated workstations, secure facilities, offline backups, or any situation where you need data to cross a physical gap without touching a wire.

---

## How it works

```
[ Sender device ]                    [ Receiver device ]

  File selected                        Camera pointed
       ↓                               at sender screen
  gzip compress                              ↓
       ↓                             Scan QR frames
  Split into chunks                  (any order, any subset)
       ↓                                    ↓
  LT fountain encode          →      Collect ~105% of chunks
       ↓                                    ↓
  Render QR stream            →      Peeling decode
  (loops forever)                           ↓
                                     Decompress + download
```

The sender loops the QR stream continuously. The receiver stops automatically once it has enough frames to reconstruct the file — no coordination required between devices.

---

## Why fountain coding

Most QR file transfer tools send chunks in a fixed sequence. Miss one frame and the transfer stalls. This project uses **LT fountain codes** — the same technique used in satellite broadcasting — which means:

- Every QR frame is equally useful
- Missed or out-of-order frames don't matter
- No retransmit, no acknowledgement, no two-way channel needed
- The receiver finishes whenever it has collected enough, regardless of which frames those were

---

## Specs

| | |
|---|---|
| Effective speed | ~14 KiB/s (text), ~5 KiB/s (already compressed) |
| Recommended file size | Under 5 MB |
| Frame loss tolerance | Up to 50%+ |
| QR frame rate | 12.5 per second |
| Bytes per frame | 400 (raw) |
| Compression | gzip level 9 |
| Works offline | Yes — once loaded, no internet needed |
| Encryption | None — encrypt the file first if needed |

---

## Tech stack

| Layer | Tool | Version |
|---|---|---|
| QR rendering | qrcode-generator | 1.4.4 |
| QR scanning | zbar-wasm | 0.11.0 |
| Compression | pako | 2.0.3 |
| Fountain code | Custom LT (vanilla JS) | — |
| Font | DM Sans + DM Mono | Google Fonts |
| Hosting | Netlify static site | — |
| Build | None — pure HTML/CSS/JS | — |

**Not WebRTC. Not a network protocol.** The transport layer is light — a screen on one side, a camera on the other. No IP stack, no signalling server, no radio frequency.

---

## Usage

1. Open the site on both devices — they don't need to be on the same network or any network
2. On the sending device → **Transmitter** → choose file → Start
3. On the receiving device → **Receiver** → Start → point camera at sender's screen
4. Hold steady at 20–40 cm. File downloads automatically when decoding completes

---

## Deploy to Netlify

No build step required — deploy the folder directly (drag-and-drop).

1. Sign in at [app.netlify.com](https://app.netlify.com)
2. Unzip this archive
3. Drag the entire folder onto the Netlify drop zone
4. Netlify assigns a live URL in ~10 seconds

HTTPS is automatic — required for camera access on the receiver page.

---

## Limitations

- No encryption — sensitive files should be encrypted before transfer
- Practical ceiling ~5 MB before transfer time becomes impractical
- Requires camera access (HTTPS — provided automatically by Netlify)
- Throughput depends on camera quality, screen brightness, and ambient light
