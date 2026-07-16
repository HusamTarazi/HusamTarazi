# RECON-32 — ESP32 Handheld Security-Research Multitool: Build Guide

> For educational and **authorized-testing** use only. Read
> [`safety-and-legal.md`](safety-and-legal.md) before you build or operate this.

A pocketable, 3D-printed ESP32 device running the open-source
[Bruce firmware](https://github.com/pr3y/Bruce) — a hands-on lab for learning
WiFi, Bluetooth/BLE, sub-GHz RF, NFC/RFID and IR on hardware you own.

- **Build time:** 1-2 hrs (Path A) to 3-6 hrs (Path B)
- **Cost:** ~$28 to ~$140 depending on modules
- **Skills:** optional-to-moderate soldering, basic 3D printing

---

## 0. Legal & ethical ground rules (read first)

Operate the radios **only** on hardware and networks you own or are explicitly
authorized to test. WiFi deauth, RF/Bluetooth jamming and flooding are
denial-of-service attacks and are illegal in most countries (the US FCC bans
jammers outright); unauthorized network access violates the CFAA and
equivalents. Cloning others' cards or intercepting private traffic is a
separate offense. **Prefer receive-only learning** — you can learn a huge
amount just by listening to your own devices. Full detail in
[`safety-and-legal.md`](safety-and-legal.md).

---

## 1. Choose a build path

**Path A — LilyGo T-Embed CC1101 (recommended, least soldering).** All-in-one
board: screen, LiPo, rotary encoder, IR, and a built-in CC1101 sub-GHz radio.
Bruce supports it directly — you mostly print a case and flash. ~$45-65.

**Path B — ESP32-S3 / "Cheap Yellow Display" (CYD) + modules (most modular).**
Start from a bare board and wire each radio yourself. Cheapest entry (a CYD is
~$12-18) and the most to learn. ~$28-90.

---

## 2. Bill of materials

See [`bom.csv`](bom.csv) for the full machine-readable list with prices and
sources. Summary:

- **Core (pick one):** LilyGo T-Embed CC1101 *or* ESP32-2432S028R (CYD) *or* ESP32-S3 DevKit (+ SPI TFT).
- **Radios:** CC1101 (sub-GHz), PN532 (NFC/RFID), NRF24L01+ (2.4GHz), IR LED + TSOP receiver, optional GPS, antennas.
- **Power:** LiPo/18650, TP4056 USB-C charger, SPDT switch, microSD.
- **Hardware:** dupont/JST wires, perfboard, M2/M3 heat-set inserts + screws, 6x3mm magnets.

**Sourcing:** AliExpress is cheapest (2-4 wk); Amazon fastest; Mouser/DigiKey
for genuine chips; buy LilyGo/M5Stack boards from their official stores.

**Tools (you already have):** soldering iron, flush cutters, tweezers,
multimeter, 3D printer. Flux and a PCB vise help.

---

## 3. Assembly (do it in this order)

1. **Bench-test the bare board.** Plug into USB with a **data** cable; confirm a serial port appears. Flash Bruce now for a known-good baseline (Section 4).
2. **Plan the bus & pin map.** SPI is shared (CC1101, NRF24, display, SD), each with its own CS; PN532 on I2C; GPS on UART. See [`wiring-pinmap.md`](wiring-pinmap.md). Write your map down.
3. **Solder the SPI radios (CC1101 + NRF24).** Mount on a perfboard hat; share SCK/MOSI/MISO, separate CS lines. CC1101 on **3.3V only**. Add a 10-100µF cap across the NRF24's VCC/GND. Attach antennas before powering.
4. **Wire NFC (I2C) + IR.** Set PN532 DIP to I2C; connect SDA/SCL/3V3/GND. Drive the IR LED via an NPN transistor; wire the TSOP receiver to your IR-RX pin.
5. **Power.** If the board lacks charging, use a TP4056 protected charger + SPDT switch on the battery positive. Mind polarity; insulate the cell.
6. **Dry-fit, then close the case.** Test-fit before final assembly, set heat-set inserts with the iron, route antennas, seat magnets, screw together. Final full-feature boot test before sealing.

Verify each stage before moving on — one module at a time is far easier to debug.

---

## 4. Flash the Bruce firmware

Use **desktop Chrome or Edge** (Web Serial). Firefox/Safari won't work.

1. Open the official web flasher: <https://bruce.computer/flasher>
2. Connect over USB-C, click **Connect**, pick the port. Install CP210x/CH340 drivers if none appears.
3. If flashing fails, force bootloader: hold **BOOT**, tap **RST/EN**, release BOOT.
4. Select your **exact device** (T-Embed CC1101, CYD-2432S028, generic ESP32-S3).
5. Click Install; wait 1-3 min; don't unplug.
6. Reboot, open **Config**, and set your **pin map / RF Config** to match your wiring.
7. Verify each module: WiFi scan, BLE scan, sub-GHz (CC1101 detected), NFC read your own card, IR capture a remote.

Update later by re-running the flasher or flashing a release `.bin` with `esptool`.

---

## 5. Print the enclosure

- **STLs:** search Printables for "T-Embed CC1101" (great shells with antenna holes, power-switch mods, GPS room). For the CYD/devkit routes search Thingiverse/Yeggi for "ESP32 CYD case". Or design a simple parametric case yourself.
- **Settings:** PETG or PLA+, 0.2mm layers, 3 walls, 20-30% infill, tree supports only on port cutouts. TPU 95A for flexible buttons/grommets. Heat-set M2/M3 inserts at ~200°C.
- **Tip:** print the case *after* dry-fitting your electronics so the cutouts match your board revision and module stack.

---

## 6. Resources

- Bruce firmware: <https://github.com/pr3y/Bruce>
- Bruce wiki: <https://wiki.bruce.computer/>
- Web flasher: <https://bruce.computer/flasher>
- ESP32 Marauder (alternative WiFi firmware): <https://github.com/justcallmekoko/ESP32Marauder>
- Learn the fundamentals legally: 802.11/WPA2 basics, TryHackMe, Hack The Box, CTFs, RTL-SDR (receive-only), and your own home lab.
