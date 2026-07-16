# Build Your Own Flipper Zero — The Complete DIY Guide

> **Goal:** Build a device as close as possible to a real Flipper Zero — same MCU
> (STM32WB55), same radios, running *real* Flipper firmware — from off-the-shelf
> modules you solder together yourself.
>
> **This is the "true clone" path.** It is the hardest of the DIY routes, but it
> gets you an actual Flipper-compatible device, not just a lookalike. This guide
> assumes you have a soldering iron + basic soldering experience and access to a
> 3D printer (you told me both). It explains everything else from scratch.

---

## 0. Read this first (5 minutes) — expectations & the law

**What you are building.** A handheld "hacker multi-tool." A real Flipper Zero can:

- **Sub-GHz radio** (300–928 MHz) — read/replay simple remotes (garage doors, some
  gates), talk to 433 MHz sensors. *(You must own or have permission for anything
  you transmit to.)*
- **125 kHz RFID** — read/emulate low-frequency access cards/fobs.
- **13.56 MHz NFC** — read/emulate NFC cards.
- **Infrared** — universal TV/AC remote, learn & replay IR codes.
- **iButton / 1-Wire** — read Dallas keys.
- **GPIO + BadUSB** — act as a USB keyboard, drive electronics, learn embedded dev.

**Honest reality check.** A DIY build will get Sub-GHz, IR, iButton, GPIO/BadUSB,
display and buttons working well. **NFC (13.56 MHz) and especially 125 kHz RFID are
the hardest parts** — the real Flipper uses a custom analog front-end for 125 kHz
that off-the-shelf modules don't perfectly replicate, and NFC support in the DIY
firmware forks is sometimes partial or fiddly. Budget extra patience (and expect to
possibly leave one of those two for "version 2"). Everything else is very doable.

**Cost.** With the good components this guide recommends and a case you print
yourself, expect roughly **$70–120** in parts, plus tools you may already own.
(A retail Flipper is ~$169–199, so you save money *and* learn 10× more.)

**Time.** Realistically a few weekends: ~1 evening to gather/plan, 1–2 evenings to
solder, 1 evening to flash and debug, then ongoing tinkering.

### ⚠️ Legal & safety — please actually read this

- **Only transmit / test on devices and networks you own or have explicit written
  permission to test.** Replaying a neighbor's garage remote, cloning access cards
  you don't own, or jamming/transmitting on licensed bands can be a **crime** in
  most countries. Radio spectrum is regulated (FCC in the US, similar bodies
  elsewhere) — even *transmitting* on some Sub-GHz frequencies without a license is
  illegal regardless of intent.
- Treat this as a **learning tool for your own hardware.** That's what makes it fun
  and legal.
- **Electrical safety:** you'll work with a Li-ion battery. Never short it, never
  charge an unprotected cell without proper charging circuitry, and use a cell with
  a built-in protection board (BMS) if unsure. Soldering iron tips reach 350 °C —
  work in a ventilated space and don't breathe flux fumes.

---

## 1. The landscape — other devices like Flipper Zero (for context)

You asked about alternatives that "do the same thing and more." Here's the map, so
you understand where the true-clone build sits:

| Option | What it is | vs. Flipper |
|---|---|---|
| **True clone (this guide)** — FCFZ / "Cheap DIY Flipper" | STM32WB55 + CC1101 + ST25R3916 + SPI LCD running real Flipper/**Momentum** firmware | Closest possible; fully compatible with the Flipper app ecosystem |
| **ESP32 + Bruce firmware** | A cheap ESP32 board flashed with [Bruce](https://bruce.computer/) | Easier & cheaper; *stronger* WiFi/BLE attacks + BadUSB + IR; Sub-GHz/RFID via add-on modules. Not "a Flipper," different firmware |
| **ESP32 Marauder** | WiFi/BLE recon firmware | Beats Flipper at 2.4 GHz WiFi sniffing/deauth/PMKID |
| **M5Stack Cardputer** (~$30) | Pre-made ESP32-S3 pocket computer, runs Bruce | Buy-and-flash, almost no soldering — the easiest "Flipper-like" device |
| **ESP32-DIV V2** | Fully open-source multi-band toolkit (PCB + case + firmware) | Community "Flipper killer," everything open + 3D-printable |

**Why the true clone for you:** you specifically wanted "as close as possible," you
have soldering skills and a 3D printer, and you're getting into embedded systems —
this build teaches you the most (SPI/I2C buses, an STM32 MCU, firmware flashing, PCB
thinking). Keep the ESP32/Bruce route in your back pocket as an easy, cheap second
project — it genuinely does *more* on WiFi.

---

## 2. How a Flipper is built — the architecture (understand before you buy)

The Flipper is one **brain** (a microcontroller) talking to several **radio/IO
modules** over two shared communication buses. Learning this now makes the wiring
step obvious instead of scary.

```
                         ┌───────────────────────────┐
                         │   STM32WB55  (the brain)   │
                         │  ARM Cortex-M4 MCU + BLE   │
                         └─────────────┬─────────────┘
        SPI bus (shared) ──────────────┼────────────── I2C bus (shared)
        (SCK/MOSI/MISO + a             │               (SCL/SDA, 2 wires,
         separate CS per device)       │                many devices, addressed)
   ┌───────────┬───────────┬───────────┼───────────┐        │
   │           │           │           │           │        │
[CC1101]   [ST25R3916]  [SPI LCD]   [SD card]  [buttons via   ...(optional sensors)
 Sub-GHz     NFC        128x64      storage    shift register)
 radio       13.56MHz   display
   │
 (GPIO-driven extras, one wire each)
   ├─ IR receiver (TSOP) + IR LED (transmit)
   ├─ iButton / 1-Wire
   ├─ Piezo speaker (PWM)
   └─ 125 kHz RFID antenna + analog front-end  ← the tricky one
```

**Two key ideas:**

1. **SPI** is a fast bus for the "big" peripherals (radio, NFC, screen, SD). They
   *share* three wires (SCK clock, MOSI data-out, MISO data-in) and each gets its
   own **Chip Select (CS)** line so the MCU can talk to one at a time.
2. **I2C** is a slower 2-wire bus (SCL clock, SDA data). Multiple chips share it and
   each has an address. (Some DIY firmware forks use I2C for the screen and a button
   expander instead of a shift register — more on firmware choice below.)

**Everything else** (IR, speaker, iButton) is just a single GPIO pin toggling.

> 📌 **The single most important rule of this whole build:** the wiring must match
> the exact pin map that your chosen **firmware fork** expects. The firmware has the
> pins hard-coded. You don't get to pick pins freely — you copy the firmware's map.
> That's why Section 6 (firmware) comes *before* final wiring.

---

## 3. Bill of Materials (BOM) — exactly what to buy

Two module ecosystems exist for this build. **This guide follows the "FCFZ / Cheap
DIY Flipper" (Momentum firmware) hardware set**, because it runs *real* Flipper
firmware and matches the real device most closely. Where a part is optional or
"advanced," it's marked.

### Core (required)

| # | Part | Recommended spec | Why | ~Price |
|---|---|---|---|---|
| 1 | **STM32WB55 dev board** | **WeAct Studio STM32WB55CGU6** board | The brain — *same MCU family as a real Flipper*. WeAct board is the community standard | $12–18 |
| 2 | **Sub-GHz radio** | **CC1101 module** (the small green one w/ SMA or spring antenna, 433 MHz) | Exactly what the Flipper uses for Sub-GHz | $3–6 |
| 3 | **Display** | **128×64 SPI LCD**, ST7565 / ST756x controller (monochrome, the "Nokia-ish" look). 8-pin SPI version preferred | Matches Flipper's screen controller family | $4–8 |
| 4 | **microSD module + card** | SPI microSD breakout + 8–32 GB card | Firmware stores apps, databases, captures here | $3 + $5 |
| 5 | **Buttons** | 5–6 tactile push buttons (Up/Down/Left/Right/OK/Back) | The D-pad | $2 |
| 6 | **Button shift register** | **SN74HC165N** (PISO shift register) + a few resistors/diodes | Lets many buttons use few MCU pins, like the real Flipper. *Optional* but recommended for compatibility | $1 |
| 7 | **Prototyping** | Perfboard/protoboard + male/female header pins + jumper/hookup wire | To mount and connect everything | $8 |
| 8 | **Battery** | **Li-ion cell with protection (BMS)**, e.g. 3.7 V 1000–2000 mAh pouch, *or* a TP4056-protected 18650 holder | Portable power | $6–10 |
| 9 | **Charger/boost** | **TP4056** charging module (USB-C version) + optional MT3608 boost | Safe charging of the Li-ion | $2 |

### Radios / sensors (add the ones you care about)

| # | Part | Spec | Notes | ~Price |
|---|---|---|---|---|
| 10 | **NFC (13.56 MHz)** | **ST25R3916** module (e.g. from Elechouse) | *Same chip as a real Flipper.* Hardest to get fully working in DIY firmware — treat as advanced | $12–20 |
| 11 | **IR receiver** | **TSOP38238** (or VS1838B) 38 kHz demodulator | Receives/"learns" IR remotes | $1 |
| 12 | **IR transmitter** | IR LED (940 nm) + NPN transistor (e.g. 2N2222) + resistor | Sends IR (must be transistor-driven, not straight off a pin) | $1 |
| 13 | **Speaker** | Small **piezo buzzer** (passive) | Tones/feedback, driven by PWM | $1 |
| 14 | **iButton / 1-Wire** | A single contact pad/pin + 1 resistor | Reads Dallas keys | <$1 |
| 15 | **125 kHz RFID** | LF antenna coil + analog front-end (**advanced/experimental**) | The real Flipper uses a custom analog stage; DIY LF is finicky. Consider skipping for v1 | $5+ |

### Nice-to-have

- **RGB status LED** (common-anode) + 3 resistors — Flipper's notification LED.
- **Vibration motor** + small N-channel MOSFET + flyback diode — haptics.
- **INA219** I2C module — battery voltage/current monitoring.

### Tools (you have soldering + a 3D printer; here's the full checklist)

- Temperature-controlled **soldering iron** (~$25+), **60/40 or lead-free solder**,
  **flux**, **desoldering braid**, brass tip cleaner.
- **Multimeter** (continuity + voltage) — *essential* for debugging; ~$15.
- Fine **tweezers**, **flush cutters**, **helping-hands / PCB holder**.
- **USB cable** (data-capable — many cheap cables are charge-only!) matching your
  board's connector (usually USB-C on the WeAct board).
- A Windows PC is easiest for flashing (STM32CubeProgrammer + Zadig). macOS/Linux
  work too but the OTP step is smoothest on Windows.
- **Filament** for the case (PLA or PETG). PETG is a bit tougher for a pocket device.

> 💡 **Buying tip:** order 2× of the cheap/fragile bits (CC1101, LCD, shift
> register). They're a couple of dollars and you *will* cook or crack one while
> learning. Buy from reputable sellers; ultra-cheap CC1101s are sometimes off-freq.

---

## 4. Plan your build order (do this before touching the iron)

1. **Flash & boot the bare MCU first.** Get the STM32WB55 board powered and talking
   to your PC *before* wiring anything. A working "blink"/bootloader connection
   proves your board + cable + drivers are good.
2. **Add the screen + buttons next.** As soon as you have a display and a D-pad, you
   have a *usable* device and instant visual feedback for everything after.
3. **Add SD card**, then **CC1101 (Sub-GHz)** — the most satisfying working radio.
4. **Add IR, speaker, iButton** — easy single-pin wins.
5. **Attempt NFC**, then **125 kHz RFID** last (the hard ones).
6. **Battery + case** once the electronics are proven on the bench.

Building in this order means every step gives you a working, testable device instead
of one giant "does nothing, why?" at the end. This staged approach is the #1 habit
that separates finished projects from drawer projects.

---

## 5. Firmware choice — decide this now (it dictates your wiring)

Because the pin map is baked into firmware, pick your firmware **before** final
wiring. For the true-clone build you have two realistic choices:

- **Momentum (FCFZ fork)** — a build of the popular *Momentum* Flipper firmware
  patched for DIY hardware. **Recommended** for "closest to real Flipper": you get
  the real Flipper UI, app ecosystem, and can install `.fap` apps. This is what the
  "Fu**ing Cheap Flipper Zero" (FCFZ) project uses.
- **A community STM32WB55 DIY firmware** (e.g. the `diy_flipper_zero` project) — a
  more "learning-oriented" fork that uses an I2C OLED + MCP23017 button expander.
  Easier to wire, but *not* fully Flipper-compatible and NFC often non-functional.

**This guide targets the Momentum / FCFZ path.** Get the exact firmware and, crucially,
its **wiring diagrams** (they are the source of truth for pins) from these repos:

- `github.com/GthiN89/FuckingCheapFlipperZero-DIY-Flipper-zero-The-real-on` — the
  "real one," off-the-shelf modules, Momentum optimized to run on them.
- `github.com/Magnowz/Flipper-Diy` — detailed **wiring diagrams** (SD, buttons with
  & without the SN74HC165N, 8-pin and 14-pin display variants, CC1101) + flashing
  instructions. **Download these diagram images — you will wire directly from them.**

> 📌 **Action:** open both repos, read their README, and save their wiring diagram
> images to your phone/PC. When this guide and a repo diagram disagree on a pin,
> **the repo diagram for your exact firmware version wins.**

---

## 6. Pin map reference (verify against your firmware's diagrams!)

Below is the FCFZ/Momentum-style mapping reported by the DIY projects. **Use it to
understand the wiring and to sanity-check, but always confirm each pin against the
wiring diagram shipped with the firmware version you flash** — forks differ, and a
wrong CS pin just means "module doesn't respond."

### Shared SPI bus (CC1101, NFC, LCD, SD all share these 3)

| Signal | STM32WB55 pin |
|---|---|
| SPI SCK (clock) | **PB3** |
| SPI MOSI (data out) | **PB5** |
| SPI MISO (data in) | **PB4** |

### Chip-Select (one unique pin per SPI device)

| Device | CS pin |
|---|---|
| Display (LCD) CS | **PA3** |
| CC1101 (Sub-GHz) CS | **PA15** |
| NFC (ST25R3916) CS | **PE4** |
| microSD CS | **PA10** |

### CC1101 extra

| Signal | Pin |
|---|---|
| CC1101 GDO0 / G0 (IRQ) | **PA1** |

### Single-pin peripherals (typical DIY mapping — confirm in firmware)

| Function | Pin | Notes |
|---|---|---|
| IR receive (TSOP out) | PA0 (or per fork) | |
| IR transmit (to transistor base) | PA8 (or per fork) | **Never** drive the IR LED straight from the pin — use the NPN transistor + resistor |
| Speaker (piezo) | PB8 (PWM) | |
| iButton 1-Wire | PA3-area / dedicated pad | Some forks share/relocate; check diagram |
| UART debug TX / RX | PB6 / PB7 | Handy for serial debugging |
| USB D− / D+ | PA11 / PA12 | Handled by the board/HAL; usually just the USB connector |

### Buttons

- **With SN74HC165N shift register (recommended, Flipper-like):** buttons feed the
  shift register's parallel inputs; the register connects to the SPI/clock lines and
  a latch pin. Wire it exactly per the "buttons with SN74HC165N" diagram in the
  `Magnowz/Flipper-Diy` repo.
- **Without shift register:** each button goes to its own MCU GPIO, button → pin →
  GND, using the MCU's internal pull-ups (active-low). Simpler, uses more pins, and
  the firmware must be configured for it. The repo warns this can degrade the input
  experience vs. the shift-register method.

> ⚠️ Note the potential conflict: some *other* DIY forks (the I2C/OLED learning fork)
> use PB4 for an I2C line and PA3 for iButton. That's a **different firmware** with a
> **different map** — don't mix diagrams across firmware forks. Pick one fork, use
> only its diagram.

---

## 7. Assembly — smallest, most detailed steps

Work on the bench with everything loose first ("dead-bug"/breadboard style is fine
for testing), and only commit to soldering onto perfboard once a subsystem responds.

### Stage A — Power up the bare MCU

1. Inspect the WeAct STM32WB55 board for shipping damage; note where **BOOT0**, **3V3**,
   **GND**, **PB3/PB4/PB5**, and the CS pins are (the silkscreen labels them).
2. Solder header pins onto the board if it didn't come pre-soldered. Heat the pad and
   the pin together ~1–2 s, feed a little solder, remove — aim for a shiny cone, not
   a blob.
3. Connect the board to your PC with a **data-capable** USB cable. A power LED should
   light. Windows should chime/enumerate a device.
4. Install **STM32CubeProgrammer** (from ST) and **qFlipper** (from Flipper) on your
   PC now. On Windows, also grab **Zadig** (for USB driver swapping later).
5. **Prove the connection:** put the board in bootloader mode — hold **BOOT0 to 3V3
   (or GND, per your board's convention)** while plugging in / pressing reset — and
   confirm STM32CubeProgrammer can **connect over USB (DFU)**. If it connects and
   reads the chip ID, your toolchain + board + cable are all good. **Do not proceed
   until this works** — it's the foundation for everything.

### Stage B — Display + buttons (your eyes and hands)

6. Wire the **SPI LCD**: VCC→3V3, GND→GND, SCK→PB3, MOSI(SDA)→PB5, CS→PA3, plus the
   display's **DC/A0**, **RST**, and **backlight** pins per the repo's 8-pin display
   diagram. (LCD modules vary — match your module's silkscreen to the diagram.)
7. Wire the **buttons**: either into the **SN74HC165N** per the shift-register diagram
   (recommended), or one-per-GPIO to GND if going simple.
8. Keep a scrap of paper: **write down every wire as you place it** ("PB3 → LCD SCK").
   This log is gold when debugging.

### Stage C — SD card + Sub-GHz

9. Wire the **microSD module**: VCC→3V3 (check if your module needs 5 V — many have a
   regulator), GND→GND, SCK→PB3, MOSI→PB5, MISO→PB4, CS→PA10.
10. Wire the **CC1101**: VCC→3V3 (**not 5 V** — CC1101 is 3.3 V!), GND→GND, SCK→PB3,
    MOSI→PB5, MISO→PB4, CS→PA15, **GDO0→PA1**. Attach its antenna (433 MHz spring or
    SMA whip). *Never power a radio without its antenna attached when transmitting.*

### Stage D — IR, speaker, iButton (single-pin wins)

11. **IR receiver (TSOP38238):** OUT→(IR-RX pin), VCC→3V3, GND→GND. That's it —
    demodulation is built in.
12. **IR transmitter:** MCU IR-TX pin → resistor (~220 Ω) → **base** of NPN transistor;
    IR LED anode → 3V3 through a current-limiting resistor, cathode → transistor
    collector; emitter → GND. This lets a weak GPIO switch a bright IR LED. **Do not**
    connect the IR LED directly to a GPIO.
13. **Speaker (passive piezo):** one leg → speaker pin (PB8), other leg → GND. Firmware
    drives it with PWM to make tones.
14. **iButton pad:** a single exposed contact + the firmware's 1-Wire pin, with a
    pull-up resistor as the diagram specifies.

### Stage E — Advanced radios (attempt after the basics boot)

15. **NFC (ST25R3916):** wire per its module's pinout to the SPI bus with CS→PE4. Keep
    the antenna coil away from metal and other boards. Expect to spend real time here —
    NFC is the fussiest subsystem in DIY firmware. If it doesn't work, don't let it
    block the rest of the build.
16. **125 kHz RFID (optional/experimental):** this is the hardest. The real Flipper uses
    a tuned LF antenna + custom analog amplifier/comparator front-end. Off-the-shelf LF
    modules rarely drop in cleanly. Treat this as a stretch goal / "v2." It's completely
    fine to ship a device without 125 kHz LF and add it later.

### Stage F — Power

17. Wire the **TP4056** charging module between the USB-C input and the Li-ion cell
    (**B+ / B−** to the cell, **OUT+ / OUT−** to your circuit). Use a **protected**
    cell or a TP4056 board with built-in protection (the "TP4056 + DW01" version).
18. If your modules need a stable 3.3 V and your battery is 3.7 V nominal, the WeAct
    board's onboard regulator generally handles logic; confirm each module's voltage
    needs. Add the MT3608 boost only if a specific part needs 5 V.
19. **Double-check polarity with your multimeter before connecting the battery.** A
    reversed Li-ion connection can destroy modules instantly.

> 🔎 **After each stage, before moving on:** run a **continuity test** with your
> multimeter (probe each wire end-to-end, and check that adjacent pins are *not*
> shorted). Five minutes of beeping now saves hours of "why is it dead" later.

---

## 8. Flashing the firmware (the software brain transplant)

The Flipper firmware needs a one-time **OTP** (One-Time Programmable) config written
to the MCU, then the main firmware flashed. **The OTP step writes memory that cannot
be rewritten — go slow and double-check values.**

> ⚠️ **Do not disconnect USB during any programming step.** A mid-write disconnect,
> especially during OTP, can brick the board.

### Step 1 — Generate the OTP file

1. Get the firmware's **OTP utility** (ships with the FCFZ/Momentum firmware, e.g. in
   a `mics`/`misc` folder in the repo).
2. Fill in the fields exactly as the repo instructs — the DIY projects use:
   **Version: 12 | Firmware: 7 | Body: 9 | Connection: 6**.
3. Set the **display type** to your LCD (e.g. "mgg") and pick a **region**
   (`us_ca_au`, `en_ru`, `jp`, or `world`).
4. Set a **device name** (max 8 characters).
5. Export/generate the **`.bin`** OTP file.

### Step 2 — Write the OTP (one time only)

1. Put the board in bootloader mode: **hold BOOT0 to ground**, then connect USB.
2. Open **STM32CubeProgrammer**, choose **USB** connection, click **Connect**.
3. Load your generated OTP **`.bin`** file.
4. Set the **start address** to **`0x1FFF7000`**.
5. Click **Start Programming**. Verify success. **Done — never write OTP again.**

### Step 3 — Flash the main firmware

1. **Remove the microSD card** first (prevents flashing errors).
2. Connect the board and open **qFlipper**.
3. Choose **Install from file** and select the modified firmware **`.dfu`** file from
   the repo.
4. If qFlipper can't see the board on Windows, use **Zadig** to install/select the
   **WinUSB** (or "USB Serial") driver for the device, then retry.
5. Let it flash and reboot. You should see the Flipper boot animation / dolphin on
   your LCD. 🎉

### Step 4 — Prepare the microSD card

1. Format the card **FAT32**.
2. Copy the firmware's recommended **databases and folders** onto it (Sub-GHz freq
   lists, IR remote DB, apps, etc.) per the repo instructions.
3. Insert the card and reboot. The device should now find its apps and databases.

---

## 9. First boot & testing each subsystem

Test in the same order you built, so a failure points at the last thing you added:

1. **Screen + buttons:** navigate the menus. Sluggish/ghost inputs? Revisit the button
   wiring / shift-register lines.
2. **SD card:** does the firmware list apps and databases? If "SD error," recheck CS
   (PA10), MISO (PB4), and card format.
3. **Sub-GHz:** open the Sub-GHz app, **Read** on 433.92 MHz near a cheap 433 remote
   you own. Seeing signal = CC1101 is alive. (Only transmit to your *own* gear.)
4. **Infrared:** use **Learn** to capture your TV remote, then replay it at the TV.
5. **iButton / speaker / LED:** quick functional checks in their apps.
6. **NFC:** try reading an NFC tag you own. Partial/no function is common in DIY —
   note it and iterate.
7. **125 kHz:** if you attempted it, expect to tune. If you skipped it, you're not
   missing the "core" experience.

**If a whole subsystem is dead:** 90% of the time it's (a) a swapped MISO/MOSI, (b) a
wrong or shorted CS pin, (c) a 3.3 V module accidentally on 5 V (or vice-versa), or
(d) a cold solder joint. Multimeter + the wiring log will find it.

---

## 10. The 3D-printed case (you have a printer!)

1. Find a case model: search **Printables / Thingiverse / GrabCAD** for "DIY Flipper
   Zero case," "FCFZ case," or "STM32WB55 Flipper case," and check the firmware repos'
   own files — the FCFZ projects often include STLs or STEP files matched to their
   board layout.
2. **Measure your actual assembled stack** with calipers (or a ruler) — DIY module
   heights vary, so verify the case's internal dimensions fit *your* build before a
   long print.
3. Print in **PLA** (easy) or **PETG** (tougher, better for a pocket device). Suggested:
   0.2 mm layer height, 3+ walls, 20–30% infill for a sturdy shell.
4. Leave/print **openings** for: USB-C port, SD slot, IR window (front), antenna,
   GPIO header, and the button pad. Add a little clearance (0.2–0.4 mm) around ports.
5. If no case fits, model a simple sandwich enclosure in **Tinkercad** (free,
   beginner-friendly) or **FreeCAD** — a great next embedded/CAD skill to pick up.

---

## 11. Troubleshooting cheat-sheet

| Symptom | Likely cause | Fix |
|---|---|---|
| PC doesn't see board at all | Charge-only USB cable; no bootloader mode | Use a data cable; hold BOOT0 correctly while plugging in |
| STM32CubeProgrammer won't connect | Wrong driver / not in DFU | Enter bootloader; install driver via Zadig |
| Screen blank but board boots | LCD CS/DC/RST miswired; wrong display variant in OTP | Recheck LCD diagram; verify display type in OTP |
| One SPI device dead, others fine | Wrong/shorted CS pin for that device | Reseat that CS wire; continuity-test it |
| All SPI dead | SCK/MOSI/MISO swapped or shorted | MISO↔MOSI is the classic swap — try swapping |
| CC1101 reads nothing | 5 V instead of 3.3 V; no antenna; off-freq clone | Power at 3.3 V; attach antenna; try your spare CC1101 |
| Buttons erratic/ghosting | Shift-register wiring; missing pull-ups | Follow the SN74HC165N diagram exactly |
| NFC won't work | DIY firmware limitation / antenna placement | Move antenna away from metal; accept partial support in v1 |
| Random resets | Weak battery/brown-out; loose power wire | Check battery + solder joints on power rails |
| Board bricked after OTP | USB disconnected mid-write | This is why we said don't disconnect — a fresh MCU board may be needed |

---

## 12. You're into embedded systems now — where to go next

You picked a genuinely deep first project. To turn this into real skill:

- **Learn the STM32 basics:** install **STM32CubeIDE**, blink an LED, then read a
  button, then talk to an I2C sensor. Understanding the MCU makes the Flipper firmware
  stop being magic.
- **Learn the buses hands-on:** SPI and I2C are everywhere in embedded. This build is
  a live example of both.
- **Read the real firmware:** the Flipper firmware is open source (GPL). Browse
  `applications/`, `furi/` (its little OS/kernel), and `targets/` to see how the pin
  map (`furi_hal_resources.h`) and apps are structured. Try building it with the `fbt`
  tool and writing a tiny `.fap` app.
- **Do the easy second build:** flash **Bruce** onto a cheap **ESP32** or an **M5Stack
  Cardputer** for a low-stakes win and a different architecture (Espressif/Arduino
  ecosystem vs. STM32).
- **Get a logic analyzer** (~$10 clone) — *seeing* SPI/I2C traffic is a superpower for
  debugging.

---

## 13. Sources & links

- Official Flipper Zero schematics & hardware docs — https://docs.flipper.net/zero/development/hardware/schematic
- FCFZ "the real one" (off-the-shelf modules, Momentum) — https://github.com/GthiN89/FuckingCheapFlipperZero-DIY-Flipper-zero-The-real-on
- Detailed wiring diagrams + flashing (Flipper-Diy) — https://github.com/Magnowz/Flipper-Diy
- STM32WB55 DIY firmware & pin-mapping (learning fork) — https://github.com/lamtranBKHN/diy_flipper_zero
- FCFZ project write-up — https://www.hackster.io/zst123/fcfz-fully-compatible-flipper-zero-e686ba
- Fu**ing cheap Flipper Zero (Hackaday) — https://hackaday.io/project/203021-fug-cheap-flipper-zero-fcfz
- Flipper Zero (Wikipedia, background) — https://en.wikipedia.org/wiki/Flipper_Zero
- Bruce firmware (for the easy ESP32 second build) — https://bruce.computer/
- ESP32 Marauder / ESP32-DIV V2 background — https://circuitdigest.com/news/meet-the-esp32-div-v2-the-open-source-flipper-zero-killer

---

*Built for you as a personal learning project. Verify every pin against your chosen
firmware's own wiring diagrams before soldering, and only ever transmit to or test
hardware you own or are authorized to test. Have fun — and print a spare case.* 🐬
