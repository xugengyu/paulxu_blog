---
title: "Debugging Transceiver Impairments on HackRF"
date: 2025-12-24
tags: notes rf hobby dsp communication
---

## HackRF Block Diagram

Has can be seen from the block diagram below, the HackRF uses a dual-conversion architecture.

The complex baseband (**BB**) signal from the MAX5864 is upconverted to (or downconverted from) an intermediate frequency (**IF**) of 2170 - 2740 MHz. The IF signal is mixed to (or from) radio frequency (**RF**) using the RFFC5072.

<p align="center">
<img src="https://paulxu.me/images/2025-12-24-hackrf-transceiver-impairments/hackrf_blockdiagram.png" alt="drawing" width="800"/>
</p>

[Source](https://hackrf.readthedocs.io/en/latest/hardware_components.html)