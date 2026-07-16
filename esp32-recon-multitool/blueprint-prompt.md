# Prompt: interactive build guide (Blueprint / Lovable)

Paste the prompt below into [Blueprint](https://blueprint.tech),
[Lovable](https://lovable.dev), or any AI website builder to generate an
interactive, app-like version of this guide (checkboxes, a live BOM cost
calculator, saved progress).

---

Build a single-page, mobile-friendly web app called "RECON-32 Build Companion" — an interactive build guide for a DIY ESP32 handheld security-RESEARCH multitool running open-source Bruce firmware. This is an educational / authorized-testing project.

Requirements:
1. Sticky top nav linking to: Ethics, Build Paths, BOM, Assembly, Flashing, 3D Printing, Resources.
2. A prominent Legal & Ethics banner at the very top: "Only operate on hardware and networks you own or are explicitly authorized to test." Make it visually distinct (warning style).
3. "Choose your build" section comparing two cards: Path A = LilyGo T-Embed CC1101 (all-in-one, least soldering); Path B = ESP32-S3 / Cheap-Yellow-Display + modules (most modular).
4. An INTERACTIVE Bill of Materials table with columns: Part, Purpose, Qty, Unit price, Where to buy. Add a quantity input per row and a live running TOTAL cost at the bottom. Group rows: Core, Radio modules, Power/hardware.
5. A step-by-step Assembly section with 6 numbered phases, each with a checkbox so the user can track progress; persist checkbox state in localStorage.
6. A Flashing section listing the 7 browser-flasher steps (bruce.computer/flasher, Chrome/Edge, connect over USB-C, bootloader, pick device, install, verify).
7. A 3D-Printing section with recommended STL sources (Printables T-Embed CC1101) and print settings (PETG, 0.2mm, 20-30% infill, heat-set inserts).
8. A Resources section linking Bruce GitHub + wiki and general wireless-security learning.

Design: dark "hardware hacker" aesthetic but NOT neon-green cliche — use a deep blue-slate background with a warm copper/amber accent and a monospace display font for headings. Clean, technical, professional. Fully responsive. Support light + dark mode.
