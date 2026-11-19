# LoRa GPS Bridge

A small web page that sends a phone's GPS position to an ESP32-S3 LoRa messenger over Bluetooth Low Energy, and mirrors the messenger's Messages (firmware 0.9.11 or later), and shows and controls the board-to-board coverage test (Range tab, firmware 0.9.18 or later).

Live page: https://kelvinpay.github.io/lora-gps-bridge/

## Use it

- **Android:** open the link in **Chrome**.
- **iPhone:** open the link in the **Bluefy** browser (Safari and Chrome on iPhone can't use Web Bluetooth).
- Tap **Connect to messenger**, allow location, and pick `LoRaMsg-XXXX`.
- Keep the page open with the screen on. It sends a fix at most every 2 s, and repeats the last one every 10 s.
- Tip: use **Add to Home screen** once while online. After that it opens like an app and works offline.

## Bluetooth protocol

Service `8f1b0001-5a6e-4c3b-9d2a-1e47c0a1b2c3`

| Characteristic | UUID | Properties | Content |
| --- | --- | --- | --- |
| Location | `8f1b0002-5a6e-4c3b-9d2a-1e47c0a1b2c3` | write, write without response | 19-byte fix below |
| Status | `8f1b0003-5a6e-4c3b-9d2a-1e47c0a1b2c3` | read, notify | short UTF-8 text from the board |

Fix packet, little-endian:

| Offset | Type | Field |
| --- | --- | --- |
| 0 | u8 | version = 1 |
| 1 | i32 | latitude × 10⁷ |
| 5 | i32 | longitude × 10⁷ |
| 9 | i16 | altitude m (−32768 = unknown) |
| 11 | u16 | accuracy in 0.1 m (65535 = 6553.5 m or worse) |
| 13 | u16 | speed cm/s (65535 = unknown) |
| 15 | u32 | fix time, Unix seconds UTC |

## Messages protocol (firmware 0.9.11+)

Service `8f1b0101-5a6e-4c3b-9d2a-1e47c0a1b2c3`

| Characteristic | UUID | Properties | Content |
| --- | --- | --- | --- |
| Send | `8f1b0102-5a6e-4c3b-9d2a-1e47c0a1b2c3` | write | `[u8 version=1][u16 client_id][UTF-8 text 1-120 bytes]` |
| Updates | `8f1b0103-5a6e-4c3b-9d2a-1e47c0a1b2c3` | read, notify | one message record per notify |
| Sync | `8f1b0104-5a6e-4c3b-9d2a-1e47c0a1b2c3` | write | `[0x01]` replays all stored messages, then an end marker |

Message record, little-endian: u8 version=1, u16 msg_id (0xFFFF = end of sync), u16 client_id, u8 direction (0 out, 1 in), u8 status (0 Sending, 1 Delivered, 2 Unconfirmed, 3 Received), u16 peer short ID, i16 RSSI dBm (-32768 unknown), u32 Unix time s (0 unknown), u8 text length, then the text.

## Range protocol (firmware 0.9.18+)

Service `8f1b0201-5a6e-4c3b-9d2a-1e47c0a1b2c3`, not advertised, needs a paired link (like Messages).

| Characteristic | UUID | Properties | Content |
| --- | --- | --- | --- |
| State | `8f1b0202-5a6e-4c3b-9d2a-1e47c0a1b2c3` | read, notify | 34-byte status, about once a second |
| Peer | `8f1b0203-5a6e-4c3b-9d2a-1e47c0a1b2c3` | read, notify | 24-byte record per board |
| Control | `8f1b0204-5a6e-4c3b-9d2a-1e47c0a1b2c3` | write | `[u8 1][u8 cmd][u32 arg]`: 1 start, 2 stop, 3 interval, 4 reply, 5 pin, 6 unpin, 7 sync, 8 power, 9 slots, 10 mark |

Byte layouts: `docs/RANGE-TEST.md` section 9 in the firmware repository. The page sends sync after subscribing.

## Files

`index.html` (the page), `manifest.json` + `sw.js` (install and offline), `icon.svg`, `icon-192.png`, `icon-512.png`. When you change a file, bump `VERSION` in `sw.js`.
