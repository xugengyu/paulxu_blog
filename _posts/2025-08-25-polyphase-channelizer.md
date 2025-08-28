---
title: "Intuitive Explanation of Polyphase Channelizer"
date: 2025-08-25
tags: dsp notes study communication
---

This post illustrates the intuition behind the polyphase channelizer.

Consider the following spectrum for a wide-band signal, which has been decomposed into its four constituent channels.
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Say we wish to extract the four narrow band signals while lowering the sampling rate. There are many ways to do this. Let us first walk through the most intuitive way, which is a straightforward decimation preceded by anti-alias filtering.

<h3>Simple Decimation Filter</h3>
For illustration, let us extract the signal on channel 3 (denoted "Signal 3" in the previos plot). 

We first apply a digital frequency shift to center the desired channel around DC. In practice, this is done by multiplying the IQ samples in time with a complex sinusoid with the proper frequency.
<iframe src="https://paulxu.me/images/2025-08-25/shifted_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Next, we apply anti-aliasing filter to remove all other channels, preventing them from folding down to DC when we down-sample later. In this case, I used a 64-tap Butterworth lowpass filter derived from the **firwin** function in the **scipy.signals** module:

<iframe src="https://paulxu.me/images/2025-08-25/lpf_taps.html" width="800" height="450" frameborder="0"></iframe>

The filtered signal looks like this:
<iframe src="https://paulxu.me/images/2025-08-25/filtered_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Finally, we down-sample the filtered signal with a decimation factor of 4.
<iframe src="https://paulxu.me/images/2025-08-25/decim_filtered_signal.html" width="800" height="450" frameborder="0"></iframe>

As an aside, checkout my post here on visualizing the down-sampling process in the frequency domain.

Notice that during decimation, we are discarding a large amount of the samples, on which we spend a considerable amount of computation effort. We had to convolve the input wide-band signal a 64-tap filter. This is a waste of computational resources. 

Furthermore, if we wish to extract the other three channels, we would need to quadruple the amount of computation. 


<h3>Polyphase Channelizer</h3>
Polyphase channelizer implements decimation filtering in a much more efficient manner than the above approach.

For illustration, let us consider the following wide-band waveform:
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal_polyphase_input.html" width="800" height="450" frameborder="0"></iframe>

To help with visualization, let us also assume that it has a purely real spectrum. The intuition we gain here can be generalized to any arbitrary waveform with complex spectra. Let us plot the spectrum of the above signal on a complex IQ plot:
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal_polyphase_input_complex.html" width="800" height="450" frameborder="0"></iframe>

First, the original signal is decomposed into 4 parallel streams with 1/4 of the sampling rate. The streams are delayed with respected to each other by one sample increments.

It may seem surprising that we down-sampled the signals without applying any anti-aliasing filter. Intuitively we would be unable to recover the desired signals on each channel, because the channels are folded on top of each other.

<iframe src="https://paulxu.me/images/2025-08-25/staggered_downsampled_streams.html" width="800" height="450" frameborder="0"></iframe>
 