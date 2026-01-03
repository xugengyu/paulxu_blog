---
title: "Getting BladeRF to work with Windows Subsystem for Linux"
date: 2026-01-03
tags: notes hobby
---

This is a short guide to getting the bladeRF SDR to work with Windows Subsystem for Linux (WSL).

## Install and configure WSL

I just installed an Ubuntu distribution directly from the Microsoft Store by searching it up. There are other ways to install WSL outlined [here](https://documentation.ubuntu.com/wsl/stable/howto/install-ubuntu-wsl2/).

Once we are inside the Ubuntu environment, we need to install [gnuradio](https://wiki.gnuradio.org/index.php/InstallingGR):

- `sudo apt-get update`

- `sudo apt-get install gnuradio python3-packaging`

## Attaching bladeRF to WSL in Windows Powershell

1. Install usbipd on Windows:

    - `winget install usbipd`

    - or use the installer [here](https://github.com/dorssel/usbipd-win/releases)

2. List all connected usb devices and note the bus id of bladeRF:

    - `usbipd list`

3. Share the bladeRF device:

    - `usbipd bind --busid <bus id>`

    - note the sharing should be persistent

4. Attach bladeRF to our Linux distribution:

    - `usbipd attach --wsl --busid <bus id>`

## Accessing bladeRF in Linux

1. Check that bladeRF is attached:

    - `lsusb`

    - `sudo bladeRF-cli -p`

3. Download the latest FPGA bitstream:

    - `wget https://www.nuand.com/fpga/hostedxA4-latest.rbf`

    - note we need to select the correct bitstream based on the version of bladeRF we are using

3. Flash the FPGA bitstream:

    - `sudo bladeRF-cli -l hostedxA4-latest.rbf`

4. Enter interactive mode of bladeRF-cli:

    - `sudo bladeRF-cli --interactive`

    - some stuff to do can be found [here](https://github.com/Nuand/bladeRF/wiki/bladeRF-CLI-Tips-and-Tricks)
