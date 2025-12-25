---
title: "Random Notes About the HackRF SDR"
date: 2025-12-24
tags: notes rf hobby dsp communication
---

## Gain Control

In **RX** mode, there are gain control at these three stages:

- RF ("amp", 0 or 11 dB)
    - Note that a value of 14 in GNURadio osmocomsdr block activates the RF amp. Any value below 14 bypasses the amp

- IF ("lna", 0 to 40 dB in 8 dB steps)

- baseband ("vga", 0 to 62 dB in 2 dB steps)

In **TX** mode, there are gain control at these two stages:

- RF (0 or 11 dB)
    - Note that a value of 14 in GNURadio osmocomsdr block activates the RF amp. Any value below 14 bypasses the amp

- IF (0 to 47 dB in 1 dB steps)

**References**

- https://github.com/greatscottgadgets/hackrf/commit/ff843584ddec86622d5d776ef3cdb7fa0ec6b700
https://osmocom.org/projects/gr-osmosdr/wiki/GrOsmoSDR#HackRF-Source-Sink