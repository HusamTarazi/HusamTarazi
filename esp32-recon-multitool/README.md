# RECON-32 — ESP32 Handheld Security-Research Multitool

An open build guide for a pocketable, 3D-printed ESP32 device running the
open-source [**Bruce** firmware](https://github.com/pr3y/Bruce). It's a
hands-on lab for learning WiFi, Bluetooth/BLE, sub-GHz RF, NFC/RFID and IR
**on hardware you own or are explicitly authorized to test**.

> ⚠️ **Read [`safety-and-legal.md`](safety-and-legal.md) first.** This is an
> educational / authorized-testing tool. Operating its radios against people,
> networks, or devices you don't own or have written permission to test is
> illegal in most countries. Prefer receive-only learning.

## What's here

| File | What it is |
|------|------------|
| [`../esp32-recon-multitool-build-guide.html`](../esp32-recon-multitool-build-guide.html) | The full **visual build guide** — open it in a browser or host it on GitHub Pages |
| [`build-guide.md`](build-guide.md) | The same guide in plain Markdown |
| [`bom.csv`](bom.csv) | Bill of materials (import into a spreadsheet or shopping cart) |
| [`wiring-pinmap.md`](wiring-pinmap.md) | Bus layout + example GPIO pin map |
| [`safety-and-legal.md`](safety-and-legal.md) | Legal & ethical ground rules |
| [`blueprint-prompt.md`](blueprint-prompt.md) | Prompt to regenerate an interactive version in Blueprint/Lovable |

## View / host the guide

- **Open locally:** download `esp32-recon-multitool-build-guide.html` and open it in any browser.
- **GitHub Pages:** enable Pages for this repo (Settings → Pages) and the HTML is served at your `github.io` URL.
- **Netlify / Vercel:** drag-and-drop the single HTML file.

## Two build paths

- **Path A — LilyGo T-Embed CC1101** (recommended, least soldering): an all-in-one
  board with screen, battery, encoder, IR and a built-in CC1101 sub-GHz radio.
- **Path B — ESP32-S3 / "Cheap Yellow Display" (CYD) + modules** (most modular):
  wire each radio yourself for maximum learning.

## Credits

Firmware: [Bruce](https://github.com/pr3y/Bruce) by pr3y and contributors.
This repository is documentation only — it ships no firmware or exploits.
