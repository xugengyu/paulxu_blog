---
title: "Building an OFDM Wireless System with GNU Radio"
date: 2025-09-01
tags: dsp notes study communication rf
---

In this post I will deep dive into how to construct an Orthogonal Frequency Division Multiplexing (OFDM) wireless system using GNU Radio and two HackRF Software Defined Radios (SDR). We will try to transfer an image over-the-air using QPSK modulation, and then demodulate the signal to recover the image.

Before attempting to transmit the image, let us first look at a simpler payload in the form of a text file. I have generated a text file with 512 ASCII characters, as shown below. (Fun fact: I generated this text using Gemini.) I added a bunch of zeros at the beginning and the end of the text file, so that we can visualize it better when it is converted to a stream of numbers.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/input_text_file.png" alt="drawing" width="400"/>
</p>

This text file is read by a "File Source" block in GNURadio, which outputs a stream of bytes (integers between 0 and 255). Note that each ASCII is one byte, or 8 bits.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/file_source.png" alt="drawing" width="400"/>
</p>

The output of the file source is fed into a "Stream to Tagged Stream" block, which adds a tag stream parallel to the main data stream (our text). In our example, we will add a tag to each packet of our input stream, with the tag value being the length of the packet. Our packet length is set to 256, meaning the whole text file can be described using two packets.

Let us visualize the tagged input stream using a "Time Sink".

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/input_tagged_stream.png" alt="drawing" width="400"/>
</p>

Since I set the File Source to repeat indefinitely, we can clearly see a periodic sequence of data samples with period 512. Each period begins and ends with the value 48, which is the ASCII value of the character "0". We can also see the tags that are added to the data stream, which occurs once every 256 samples.

