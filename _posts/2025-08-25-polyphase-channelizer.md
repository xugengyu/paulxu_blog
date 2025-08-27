---
title: "Polyphase Channelizer"
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

Next, we apply anti-aliasing filter to remove all other channels, preventing them from folding down to DC when we downsample later. In this case, I used a 64-tap Butterworth lowpass filter derived from the **firwin** function in the **scipy.signals** module.

<iframe src="https://paulxu.me/images/2025-08-25/filtered_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Finally, we down-sample the filtered signal with a decimation factor of 4.
<iframe src="https://paulxu.me/images/2025-08-25/downsampled_signal.html" width="800" height="450" frameborder="0"></iframe>

As an aside, checkout my post here on visualize the downsampling in the frequency domain.
<h3>Polyphase Filter</h3>
 