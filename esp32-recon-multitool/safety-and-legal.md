# Safety, legal & ethical ground rules

This project is for **learning** and **authorized testing** only. The radios
that let you study a protocol can also disrupt other people's devices — and
that crosses from education into a crime. Treat these rules as the real spec.

## Operate only on what you own or are authorized to test

- **Your own gear only** — your home WiFi, your own RFID fobs, your own remotes, a lab network you built.
- For anything else you need **explicit written authorization**: a signed penetration-test scope, a CTF's published rules, or a lab you were invited into.

## Transmitting can be illegal

- **WiFi deauthentication, RF/Bluetooth jamming, and flooding are denial-of-service attacks.** In the US the FCC prohibits signal jammers outright.
- **Unauthorized access** to networks or systems violates the US CFAA and equivalent computer-misuse laws worldwide.
- **Cloning someone else's** access card, or intercepting private traffic, is a separate offense.
- Transmitting on **licensed radio bands** without authorization is also regulated.

## Prefer receive-only learning

You can learn an enormous amount by *listening*:

- Sniff and inspect **your own** WiFi packets and BLE advertisements.
- Read **your own** NFC/RFID tags and study how they identify themselves.
- Wardrive to see how 802.11 beacons and channels work (passive).
- Use an RTL-SDR for receive-only RF exploration before you ever transmit.

## Electrical & battery safety

- Charge Li-ion/LiPo cells the first time on a **fireproof surface** and watch them.
- Never let solder joints touch the battery pouch; insulate with Kapton tape.
- Double-check polarity before powering anything.

## Bottom line

If a feature's only purpose is to degrade or intrude on a system you don't
control, don't use it. This documentation covers how to **build and
understand** the device; how you operate it is your responsibility.
