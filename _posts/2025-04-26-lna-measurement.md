---
title: "Amplifier Characterization Using TinySA and NanoVNA"
date: 2025-04-26
tags: notes study
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

Putting a $$50\Omega$$ termination directly at the input port of the TinySA, we measure a noise level of -169.5dBm/Hz. This value can be used to calculate the effective noise temperature of the receiver inside the TinySA. We will assume that the resistor is at the IEEE reference temperature of 290K. In practice, of course the physical temperature of our termination will be slightly different. However, small differences like this will not cause significant errors in our measurement.

On the TinySA, we measure a noise floor of $$\mathrm{-169.3dBm/Hz}$$.
Since the thermal noise floor is $$10\log_{10}(1.38\times10^{-23}\times10^{3}\times290)=\mathrm{-173.9dBm/Hz}$$, we conclude that the TinySA increased the total noise power by approximately 4.6dBm/Hz.This means the noise figure of TinySA is about 4.6dB. Alternatively, we can say it has an effective noise temperature of approximately $$290\left(10^{4.6/10}-1\right)=546.4\mathrm{K}$$. Note that the IEEE definition of noise factor (linear version of noise figure) is 

$$
F=\frac{SNR_{in}}{SNR_{out}}
$$

where $$\mathrm{SNR}_{in}$$ is the signal-to-noise ratio at the DUT input,with the noise power being the noise power of a resistor at the reference temperature $$T_0=290K$$.

Having characterize the noise properties of the TinySA, we can now measure the DUT cascaded with the internal receiver. We will then remove the added noise of the TinySA from the overall measurements, to obtain the noise figure of the DUT itself.

The gain of the DUT is 


## Use in FM Receiver
### Long Lossy Cable
