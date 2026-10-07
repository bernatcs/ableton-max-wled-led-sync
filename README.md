# Ableton → Max/MSP → WLED LED Sync

MIDI-driven RGB control for 4 independent WS2812B LED strips, each powered by its own ESP32 running WLED. Color data is automated in Ableton Live as note velocity, converted to RGB in a Max/MSP patch, and streamed to each strip over WLED's UDP realtime protocol.

## How it works

```
Ableton Live (velocity automation, 2 MIDI tracks × 16 notes)
        │ MIDI
        ▼
Max/MSP patch (max/midi_to_wled.maxpat)
  notein → scale 0-127 → 0-255 → pak R G B
  uzi 60 → fills all 60 LEDs with the same color
  prepend 2 255 → WLED DRGB protocol header (protocol=2, timeout=255s)
  sadam.udpSender → raw UDP packet
        │ UDP (DRGB)
        ▼
4× ESP32 running WLED, data pin GPIO16
  W1 → 192.168.10.10:65501
  W2 → 192.168.10.11:65502
  W3 → 192.168.10.12:65503
  W4 → 192.168.10.13:65504
        │
        ▼
4× WS2812B strips, 60 LEDs each
```

Each note's velocity drives one of 4 channels (Alpha, R, G, B) for one of the 4 strips — 16 channels total, duplicated across two MIDI tracks/ports ("to Max 1" / "to Max 2").

Inside the patch, the RGB value is repeated 60 times (`uzi 60`) and grouped into a single list of 180 numbers (60 LEDs × RGB), then prefixed with WLED's realtime protocol header before being sent as a raw UDP packet. Each WLED instance keeps its default UDP port (21324) active but also listens on a second, device-specific port (set in WLED's Sync Setup) so the four strips can be addressed independently from the same Max patch without colliding.

All 4 ESP32/WLED devices are configured identically (60× WS2812B on GPIO16, RGB mode) — only the IP and the second sync port differ, per the table above.

## Hardware

- 4× ESP32 (WiFi+BT SoC, ISM 2.4GHz 802.11 b/g/n)
- Data output on GPIO16 of each ESP32
- 4× WS2812B addressable LED strip, 60 LEDs per strip
- Firmware: [WLED](https://kno.wled.ge/)

## Software dependencies

- Ableton Live (velocity automation)
- Max/MSP 8 (patch built on 8.6.5)
- [Sadam Library](https://cycling74.com/packages/sadam-library) for Max (provides `sadam.udpSender`) — install via the Cycling '74 Package Manager

## Repo contents

- `max/midi_to_wled.maxpat` — the Max/MSP patch
- `wled/w1/cfg.json` — WLED config backup (WiFi SSID redacted). W2–W4 are identical except IP and sync port (see table above). No custom presets are used — all control is live via UDP from Max.

## Status

Reconstructed from the original Max patch and a live config backup. Ableton project file not yet recovered.
