---
title: "Building an OFDM Wireless System with GNU Radio"
date: 2025-09-01
tags: dsp notes study communication rf
---

In this post I will deep dive into how to construct an Orthogonal Frequency Division Multiplexing (OFDM) wireless system using GNU Radio and two HackRF Software Defined Radios (SDR). We will try to transfer an image over-the-air using QPSK modulation, and then demodulate the signal to recover the image.

Before attempting to transmit the image, let us first look at a simpler payload in the form of a text file. I have generated a text file with 512 ASCII characters, as shown below. (Fun fact: I generated this text using Gemini.) I added a bunch of zeros at the beginning and the end of the text file, so that we can visualize it better when it is converted to a stream of numbers.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/input_text_file.png" alt="drawing" width="500"/>
</p>

This text file is read by a "File Source" block in GNURadio, which outputs a stream of bytes (integers between 0 and 255). Note that each ASCII is one byte, or 8 bits.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/file_source.png" alt="drawing" width="500"/>
</p>

The output of the file source is fed into a "Stream to Tagged Stream" block, which adds a tag stream parallel to the main data stream (our text). In our example, we will add a tag to each packet of our input stream, with the tag value being the length of the packet. Our packet length is set to 256, meaning the whole text file can be described using two packets.

Let us visualize the tagged input stream using a "Time Sink".

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/input_tagged_stream.png" alt="drawing" width="500"/>
</p>

Since I set the File Source to repeat indefinitely, we can clearly see a periodic sequence of data samples with period 512. Each period begins and ends with the value 48, which is the ASCII value of the character "0". We can also see the tags that are added to the data stream, which occurs once every 256 samples, i.e. there is a tag for each packet.

Next, we add a cyclic redundancy check (CRC) code to all of our packets, using the "Stream CRC32" block. This block takes each of our packets, computes a 32-bit CRC code, and adds it to the end of the packet. When the data + CRC code is received later, the receiver will try to perform calculations to decide if the received data has been corrupted. There are several ways this decision can be made at the receiver. I will not go through that here.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/CRC.png" alt="drawing" width="500"/>
</p>

The output of the CRC block is shown below. Notice that each packet is 260-byte long, with the last 4 bytes being the CRC code.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/input_stream_with_CRC.png" alt="drawing" width="500"/>
</p>

With the CRC code added, we can now convert our stream of bytes into a stream of QAM symbols. For this example, we will use QPSK modulation, which maps each pair of bits to one of four possible points on the IQ (complex) plane. Before mapping to the complex plane, we first convert each byte (8 bits) into four consecutive bit pairs (which will later be mapped to 4 consecutive symbols). We do this using the "Repack Bits" block.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/pack_8bits_to_2bits.png" alt="drawing" width="500"/>
</p>

Looking at the output of this block, we see that the values are now restricted to [0,3], corresponding to {00, 01, 10, 11} in binary. Furthermore, each packet that was originally 260 samples long has now become 1080 samples long, since each of the original sample has now been split into 4 consecutive samples.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/2bit_stream.png" alt="drawing" width="500"/>
</p>

Another important step to perform before we map our data (and CRC) stream to a QPSK constellation is to generate headers for each packet. The header acts as sort of a metadata for the data packet, telling us information such as the length of the packet and the packet number, which helps the receiver reconstruct the data stream later. The header also includes a (smaller) CRC code for itself to guard against errors. For each data packet, we generate a corresponding header using the "Protocol Formatter" block. The exact format of the header depends on the header format object used.In this example, we are using the built-in header_format_ofdm class.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/input_stream_header.png" alt="drawing" width="500"/>
</p>

The output of the Protocol Formatter is 8-bit. We repack it into 1-bit binary representation, so that the header may be transmitted using BPSK modulation. Looking at the output of the Protocol Formatter, we see a stream of 48-bit long headers.

Looking at the repacked headers, we see a stream of 48-bit long packets. Notice that even though our data packets are repeated indefinitely by the file source block, the headers are never repeated. This is because the headers contain information unique to each packet, such as the packet number.

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/1-bit_headers.png" alt="drawing" width="500"/>
</p>

We now have a data stream that consists of values between [0, 3], and a header stream with values between [0, 1]. In order to transmit them using QAM, we must map them to the correct QPSK and BPSK constellation respectively. This is done using the "Chunks to Symbols" block. As can be seen below, each of these blocks takes the symbol table for the desired modulation format, and maps the input samples to a constellation point (a complex number). 

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/constellation_map.png" alt="drawing" width="500"/>
</p>

Looking at the output of the symbol maps, we see that the 0s and 1s of the header stream has been converted to -1s and 1s. The 2-bit chunks of the data stream (00, 01, 10, 11) have been mapped to the QPSK constellation points following the table [here](https://www.gnuradio.org/doc/doxygen/classgr_1_1digital_1_1constellation__qpsk.html).

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/header_payload_symbols.png" alt="drawing" width="500"/>
</p>

Next we stitch the payload stream and the header stream together using the "Tagged Stream Mux" block. This block takes two inputs sequentially back and forth, and outputs a single combined stream. This way, we have a single stream in which each payload is preceded by its corresponding header.
<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/stream_mux.png" alt="drawing" width="500"/>
</p>

Looking at the output, we see that each packet is now 1088 samples long. This is because each header packet is 48 samples long, and the payload packet is 1040 samples long. Furthermore, we see that each packet begins with a short sequence of BPSK symbols (header), which is followed by a longer sequence of QPSK symbols (payload).

<p align="center">
<img src="https://paulxu.me/images/2025-09-13-gnuradio-ofdm/muxed_header_payload.png" alt="drawing" width="500"/>
</p>
