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

## Use in FM Receiver
