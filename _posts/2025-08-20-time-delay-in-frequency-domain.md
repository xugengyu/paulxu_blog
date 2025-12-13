---
title: "Visualizing Time Delay in Frequency Domain"
date: 2025-08-20
tags: dsp notes study communication
---

This is an interactive plot I made illustrating the effect of time delay in the frequency domain.

On the left, I plot the time domain signal, which consists of a simple impulse that has been delayed by some number of samples. You can use the slider on the bottom to adjust the applied time delay.

On the right, I plot the fast Fourier Transform (FFT) of the delayed impulse. We can observe that delay essentially creates a phase rotation to the frequency domain samples. Each sample delay in time adds one phase spiral in frequency.

<iframe src="https://paulxu.me/images/20250820-delay-fft.html" width="800" height="600" frameborder="0"></iframe>

Once we go past 16 samples of delay (there are 32 samples in total), the number of spirals in the frequency domain starts decreasing. This continues until we have 31-sample delay, which has only a single phase spiral in frequency domain.

If we compare the FFT for delay=1 and delay=31, we see that they both have a single phase spiral, but with opposite orientations. This is because FFT, which is basically a discrete Fourier transform (DFT), assumes that the signal is periodic (in frequency and in time). So a delay of 31 samples is equivalent to an advance of 1 sample (or a delay of -1 sample).