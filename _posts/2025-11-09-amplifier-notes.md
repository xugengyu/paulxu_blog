---
title: "Some Notes on RF Amplifiers"
date: 2025-10-03
tags: notes study rf
---
## Memory Effects

## Linearization Techniques

- Feed-forward Linearization
- Analog Predistortion
- Digital Predistortion

## Nonlinear Distortion

Output tones in a memoryless system modeled by a third-order polynomial, when it is subject to a two-tone input.

$$y(t)= a_0+a_1x(t)+a_2x^2(t)+a_3x^3(t)$$

![alt text](./images/understanding-the-third-order-intercept-point-of-a-cascaded-system-fig2.jpg)

The amplitudes of the tones are not to scale.

### Dynamic Range (DR)

$$\mathrm{DR} = \mathrm{P}_{out,1dB}-\mathrm{N}_o$$

Where  \\(\mathrm{P}_{out,1dB}\\) is the output power at the 1dB compression point, and \\(N_o\\) is the noise floor.

Practically speaking, the usable dynamic range depends on the modulation scheme, bit rate, energy per bit, filter bandwidth, etc.

### Spurious-Free Dynamic Range (SFDR)

$$\mathrm{SFDR} = \frac{2}{3}\left(\mathrm{OIP}_3-\mathrm{N}_o\right)$$

Where  \\(\mathrm{OIP}_3\\) is output third order intercept point, and \\(N_o\\) is the noise floor.

### IM3 Distortion Power

The power in the intermodulation product is given by

$$\mathrm{P}_{IM,dBm} = 3\mathrm{P}_o-2\mathrm{OIP}_3$$

where \\(\mathrm{P}_o\\) is the output power of the fundamental component and \\(\mathrm{OIP}_3\\) is the output third order intercept point.

### Cascaded IM3 Distortion

Roughly speaking, there are two mechanisms that contribute to the IM3 power of a cascaded system:

- The IM3 gain of the first stage gets linearly amplified by the second stage.
- The fundamental output of the first stage generates IM3 in the second stage.

These IM3 signals are not random, hence we cannot just add in power. In reality, when dealing with wide band signals, the addition will be somewhere in between coherent and incoherent. We can choose the degree of coherency in our analysis based on the conservatism we want to apply.

In the worst case when the distortion components add in voltage (coherent):

$$\frac{1}{(\mathrm{OIP3}_{cas})^{1/2}}=\frac{1}{(\mathrm{OIP3}_{1}G_2)^{1/2}}+\frac{1}{(\mathrm{OIP3}_{2})^{1/2}}$$

In the best case when the distortion components add in power (incoherent):

$$\frac{1}{\mathrm{OIP3}_{cas}}=\frac{1}{\mathrm{OIP3}_{1}G_2}+\frac{1}{\mathrm{OIP3}_{2}}$$

Note that the nonlinearity of the final stage is dominant over the nonlinearity of the first stage.
