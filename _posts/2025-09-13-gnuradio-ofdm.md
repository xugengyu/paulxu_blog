---
title: "Building an OFDM Wireless System with GNU Radio"
date: 2025-09-01
tags: dsp notes study communication rf
---

In this post I will deep dive into how to construct an Orthogonal Frequency Division Multiplexing (OFDM) wireless system using GNU Radio and two HackRF Software Defined Radios (SDR). We will try to transfer an image over-the-air using QPSK modulation, and then demodulate the signal to recover the image.

Before attempting to transmit the image, let us first look at a simpler payload in the form of a text file. I have generated a text file with 512 ascii characters, as shown below.

![alt text](./2025-09-13-gnuradio-ofdm/input_text_file.png)