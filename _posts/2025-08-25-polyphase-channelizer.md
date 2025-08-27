---
title: "Polyphase Channelizer"
date: 2025-08-25
tags: dsp notes study communication
---

This post illustrates the intuition behind the polyphase channelizer.

Consider the following spectrum for a wide-band signal, which has been decomposed into its four constituent channels.
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal.html" width="800" height="600" frameborder="0"></iframe>

Say we wish to extract the four narrow band signals while lowering the sampling rate. There are many ways to do this. Let us first walk through the most intuitive way, which is a straightforward decimation preceded by anti-alias filtering.

<h3>Simple Decimation Filter</h3>
For illustration, let us extract the signal on channel 3 (denoted "Signal 3" in the previos plot). 

We first apply a digital frequency shift to center the desired channel around DC.
<iframe src="https://paulxu.me/images/2025-08-25/shifted_wideband_signal.html" width="800" height="600" frameborder="0"></iframe>

Then, we apply anti-aliasing filter to remove all other channels, preventing them from folding down to DC when we downsample later. In this case, I applied a 64-tap Butterworth lowpass filter. 

<iframe src="https://paulxu.me/images/2025-08-25/filtered_wideband_signal.html" width="800" height="600" frameborder="0"></iframe>

Lastly, we down-sample the filtered signal with a decimation factor of 4.
<iframe src="https://paulxu.me/images/2025-08-25/downsampled_signal.html" width="800" height="600" frameborder="0"></iframe>

<h3>Polyphase Filter</h3>
 