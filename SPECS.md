# Airgap QR Transfer — Specification Sheet

Fountain-coded file transfer between offline devices using only a screen and a camera.

---

## Overview

| | |
|---|---|
| What it does | Moves files between two devices with no network, cable, or Bluetooth |
| How | Sender displays a stream of QR codes; receiver reads them with its camera |
| Key property | Fountain-coded — missed frames don't matter, no frame is "required" |
| Works offline | Yes, once the page is loaded |

---

## Transfer protocol

| Spec | Value |
|---|---|
| Coding scheme | LT fountain code (Luby, 2002) |
| Frame-loss tolerance | 50%+ (decoder finishes on any sufficient subset) |
| Decode overhead | ~1.10–1.17× the number of source blocks |
| PRNG | mulberry32, seeded per symbol |
| Degree distribution | Ideal-soliton-ish, with a 6% degree-1 spike for bootstrapping |
| Integrity check | FNV-1a 32-bit hash carried in the header frame |
| Endpoint | Sender emits indefinitely; receiver stops itself when fully decoded |

---

## Encoding pipeline

| Stage | Detail |
|---|---|
| Compression | gzip level 9 (pako) |
| Source block size | 400 raw bytes |
| Framing | 1-byte type + 4-byte seed + payload |
| Header frame | type + k (u32) + chunkSize (u16) + hash (u32) + name |
| Header cadence | Emitted every 15th QR |
| Wire encoding | base64 (ASCII — survives the scanner's UTF-8 decode) |
| Characters per QR | ~540 |

---

## QR layer

| Spec | Value |
|---|---|
| QR generator | qrcode-generator 1.4.4 |
| QR scanner | zbar-wasm 0.11.0 |
| QR mode | Byte |
| Error correction | Level M (~15% recovery) |
| QR version | Auto — smallest that fits the payload |
| Frame rate | 80 ms per frame = 12.5 QR/s |

---

## Throughput

Effective speed depends on how well the file compresses.

| Source type | gzip ratio | Effective speed |
|---|---|---|
| Already compressed (jpg, mp4, zip) | ~1× | ~4.9 KiB/s |
| Office documents (docx, pdf) | ~2× | ~9.8 KiB/s |
| Text / source code | ~3× | ~14 KiB/s |
| Highly redundant text | ~5× | ~24 KiB/s |

Raw compressed-data throughput (after fountain overhead): **~4.2 KiB/s**.

---

## Maximum file size

There are two separate ceilings.

### Technical ceiling (device memory)

The receiver holds the whole file in RAM several times during reconstruction
(recovered blocks, reassembled copy, decompressed output). There is no hard cap
in the code itself — the chunk-count field alone allows 4.3 billion blocks.

| Device | Text file (3× gzip) | Already-compressed file |
|---|---|---|
| iPhone (Safari) | ~480 MB | ~267 MB |
| Android (Chrome) | ~300 MB | ~167 MB |
| Desktop browser | ~1.2 GB | ~667 MB |

### Practical ceiling (time + reliability)

Long transfers take a long time and raise the chance of a mid-stream disruption.

| File (text) | Transfer time |
|---|---|
| 1 MB | ~1 min |
| 5 MB | ~6 min |
| 10 MB | ~13 min |
| 25 MB | ~32 min |
| 50 MB | ~1.1 h |
| 100 MB | ~2.1 h |

**Recommendation:** keep files under ~5 MB. The tool can technically move hundreds
of MB, but transfer time and the need to hold the camera steady make large files
impractical.

---

## Practical operation

| Parameter | Value |
|---|---|
| Camera distance | 20–40 cm from the sender's screen |
| Best conditions | High screen brightness, steady devices, low glare |
| Sender screen | Maximise brightness; a stand helps |
| Comfortable file size | Under 1 MB |
| Workable file size | 1–5 MB |

---

## Site & deployment

| Spec | Value |
|---|---|
| Pages | `/` (landing), `/send`, `/receive` |
| Hosting | Netlify static site (drag-and-drop deploy) |
| HTTPS | Automatic — required for camera access |
| Dependencies | pako, qrcode-generator, zbar-wasm (all via CDN) |
| Offline use | Works once loaded; save the page for true airgap |

---

## Reliability test results

Full pipeline (base64 encoding + UTF-8 channel + trailing newline + simulated loss),
80 source blocks of 400 bytes:

| Frame loss | Symbols emitted | Received | Decoded | Byte-exact |
|---|---|---|---|---|
| 0% | 107 | 107 | yes | yes |
| 10% | 116 | 102 | yes | yes |
| 30% | 156 | 98 | yes | yes |
| 50% | 205 | 96 | yes | yes |

---

## Known limitations

| Limitation | Note |
|---|---|
| No encryption | Anyone who sees the screen can capture the data |
| No two-way ACK | Sender can't tell what the receiver has; it just keeps emitting |
| Memory-bound | Whole file is reconstructed in RAM — large files strain mobile devices |
| Hash not enforced | Integrity hash is carried but not verified on completion |
| Single-camera | Receiver needs a working rear camera; sender needs a bright screen |
