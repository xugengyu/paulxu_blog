---
title: "Intuitive Explanation of Polyphase Channelizer"
date: 2025-08-25
tags: dsp notes study communication
---

This post illustrates the intuition behind the polyphase channelizer.

Consider the following spectrum for a wide-band signal, which has been decomposed into its four constituent channels.
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal.html" width="800" height="500" frameborder="0"></iframe>

Say we wish to extract the four narrow band signals while lowering the sampling rate. There are many ways to do this. Let us first walk through the most intuitive way, which is a straightforward decimation preceded by anti-alias filtering.

<h3>Simple Decimation Filter</h3>
For illustration, let us extract the signal on channel 3 (denoted "Signal 3" in the previous plot). 

We first apply a digital frequency shift to center the desired channel around DC. In practice, this is done by multiplying the IQ samples in time with a complex sinusoid with the proper frequency.
<iframe src="https://paulxu.me/images/2025-08-25/shifted_wideband_signal.html" width="800" height="500" frameborder="0"></iframe>

Next, we apply anti-aliasing filter to remove all other channels, preventing them from folding down to DC when we down-sample later. In this case, I used a 64-tap finite impulse response (FIR) low-pass filter synthesized using the **firwin** function in the **scipy.signals** module:

<iframe src="https://paulxu.me/images/2025-08-25/lpf_taps.html" width="800" height="500" frameborder="0"></iframe>

The filtered signal looks like this:
<iframe src="https://paulxu.me/images/2025-08-25/filtered_wideband_signal.html" width="800" height="700" frameborder="0"></iframe>

Finally, we down-sample the filtered signal with a decimation factor of 4.
<iframe src="https://paulxu.me/images/2025-08-25/decim_filtered_signal.html" width="800" height="700" frameborder="0"></iframe>

As an aside, checkout my post here on visualizing the down-sampling process in the frequency domain.

Notice that during decimation, we are discarding a large amount of the samples, on which we have spent a considerable amount of computation resources. We had to convolve the wide-band input signal with a 64-tap FIR filter.

Furthermore, if we wish to extract the other three channels, we would need to quadruple the amount of computation. 


<h3>Polyphase Channelizer</h3>
A polyphase channelizer implements decimation filtering much more efficiently than the straightforward approach described above.

To illustrate this, let us consider the following wide-band waveform with uniform, consisting of four narrow-band signals, each with a perfectly flat spectrum:
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal_polyphase_input.html" width="800" height="500" frameborder="0"></iframe>

To aid visualization, let us assume that the signal has a purely real spectrum. The intuition developed in this case can later be generalized to arbitrary waveforms with complex spectra. Now, let us plot the spectrum of the signal on a complex IQ plane:
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal_polyphase_input_complex.html" width="800" height="500" frameborder="0"></iframe>

First, the original signal is decomposed into four parallel streams, each operating at one-quarter of the original sampling rate. These streams are staggered with respect to one another by one-sample increments.

At first glance, it may seem surprising that the signals are down-sampled without any anti-aliasing filter. Intuitively, this should make it impossible to recover the desired signals on each channel, since the spectral content would fold over and overlap.

Indeed, if we examine only a single down-sampled stream, this is exactly what happens. However, when we consider all four streams together and decompose each into contributions from the four narrow-band signals, we find that each stream contains a unique mixture of these signals, distinguished by different phase shifts. The following plot illustrates this effect. Play with the slider to visualize the decomposition of different streams.

<iframe src="https://paulxu.me/images/2025-08-25/staggered_downsampled_streams.html" width="800" height="500" frameborder="0"></iframe>
 
With each stream alone, it is indeed impossible to recover the four narrow-band signals, as expected from the Nyquist sampling criterion. However, the advantage of a polyphase channelizer is that by cleverly combining the four parallel streams, it becomes possible to cancel out the unwanted signals while preserving the one of interest.