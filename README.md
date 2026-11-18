# LoRa GPS Bridge

A small web page that sends a phone's GPS position to an ESP32-S3 LoRa messenger over Bluetooth Low Energy.

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

## Files

`index.html` (the page), `manifest.json` + `sw.js` (install and offline), `icon.svg`, `icon-192.png`, `icon-512.png`. When you change a file, bump `VERSION` in `sw.js`.
