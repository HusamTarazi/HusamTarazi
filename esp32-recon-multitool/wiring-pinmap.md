# Wiring & example pin map

## Bus layout

- **SPI bus (shared):** display, CC1101, NRF24, microSD. All share `SCK / MOSI / MISO`; each device gets its **own CS (chip-select)** line.
- **I2C:** PN532 NFC (`SDA / SCL`). Set the PN532's onboard DIP switches to **I2C** mode.
- **UART:** GPS module (`TX / RX`, crossed).
- **GPIO:** IR transmit (through an NPN transistor) and IR receive (TSOP receiver).

## Example GPIO map

> These are **example** pins. The exact free GPIOs depend on your board (CYD,
> ESP32-S3 devkit, and T-Embed all differ). Pick pins not already used by the
> screen/PSRAM, then set them in **Bruce → Config → Pins / RF Config**.

| Signal | Bus | Example GPIO | Notes |
|--------|-----|--------------|-------|
| SCK / MOSI / MISO | SPI (shared) | 12 / 11 / 13 | One bus, all SPI modules |
| CC1101 CS / GDO0 | SPI | 10 / 9 | Sub-GHz select + data |
| NRF24 CE / CSN | SPI | 4 / 5 | 2.4GHz transceiver |
| SD CS | SPI | 21 | Log storage |
| PN532 SDA / SCL | I2C | 8 / 18 | Set PN532 DIP to I2C |
| IR TX / RX | GPIO | 44 / 43 | LED via transistor; TSOP receiver |
| GPS TX / RX | UART | 17 / 16 | Cross RX-TX |

## Power notes

- Power the **CC1101 from 3.3V, never 5V**.
- The **NRF24 is noise-sensitive** — solder a 10-100µF capacitor across its VCC/GND right at the module to prevent brownouts.
- **Always attach antennas before powering up** any RF module; transmitting without one can damage the RF stage.
- Observe **LiPo polarity** — reversed will destroy the board. Insulate the cell with Kapton tape.
