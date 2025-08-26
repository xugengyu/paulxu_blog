---
title: "Polyphase Channelizer"
date: 2025-08-25
tags: dsp notes study communication
---

This post illustrates the intuition behind the polyphase channelizer.

Consider the following spectrum for a wide-band signal, which has been decomposed into its four constituent channels.
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal.html" width="800" height="600" frameborder="0"></iframe>

Say we wish to extract the four narrow band signals while lowering the sampling rate. The most straightforward method would be to:
1. Apply a digital frequency shift to center the desired channel around DC
2. Apply anti-aliasing filter to remove all other channels
3. Downsample to desired sampling rate

This approach is extremely inefficient in terms of computation. For each of the 4 channels, we need to process all 4000 samples (during Step 2),

 