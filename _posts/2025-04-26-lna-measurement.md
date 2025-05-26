---
title: "Amplifier Characterization Using TinySA and NanoVNA"
date: 2025-04-26
tags: notes
---

In this post I discuss measurement of several important parameters of RF amplifiers using the TinySA (spectrum analyzer) and NanoVNA (vector network analyzer).

## Introduction

## Equipment

#### TinySA

#### NanoVNA

## Device-Under-Test (DUT)

#### Nooelec LaNA HF
This is a 50KHz-150MHz low noise amplifier (LNA) I bought to pair with my HackRF. I like it since it can be powered via USB, which helps me keep the desktop clean while working on my projects.


## Gain 
To measure the gain of the amplifier, we can use a VNA. We first calibrate the VNA for our frequency range (See notes on VNA Calibration). The NanoVNA comes with a set of manual calibration standards for us to use.

Unlike a more sophisticated (and much more expensive) VNA, the NanoVNA only has a source at port 1 which cannot be switched to port 2. Therefore, it can only measure $$S_{11}$$ and $$S_{21}$$ at a given time. But that is not a problem, since we can manually flip the direction of the DUT to measure $$S_{22}$$ and $$S_{12}$$.

### Gain Compression

## Intermodulation Distortion

## Noise Figure
We can measure the noise figure of our amplifier with the TinySA, using the cold source method.

Putting a $$50\Omega$$ termination directly at the input port of the TinySA, we measure a noise level of -169.5dBm/Hz. This value can be used to calculate the effective noise temperature of the receiver inside the TinySA. 

Since the thermal noise floor is $$10\log_{10}(1.38\times10^{-23}\times10^{3}\times290)=\mathrm{-173.9dBm/Hz}$$, we see that the TinySA increased the noise power by approximately 4.6dBm/Hz.This means the noise figure of TinySA is about 4.6dB. Alternatively, we can say it has a noise temperature of approximately $$290\left(10^{4.5/10}-1\right)=546.4\mathrm{K}$$.

Having characterize the noise properties of the TinySA, we can now measure the DUT. We will then remove the added noise of the TinySA from the overall measurements, to obtain the noise figure of the DUT itself.

The gain of the DUT is 

Note that

## Use in FM Receiver
### Long Lossy Cable
