---
title: "Polyphase Channelizer"
date: 2025-08-25
tags: dsp notes study communication
---
The polyphase channelizer is a powerful tool used to efficiently split a wideband signal into multiple narrower channels. The plot below shows a simplified block diagram illustrating how a single channel is extracted, which we will examine first. Once we understand this process, we will modify the block diagram to extract all channels simultaneously.

<iframe src="https://paulxu.me/images/2025-08-25/block_diagram.svg" width="850" height="500" frameborder="0"></iframe>

This post illustrates the intuition behind the polyphase channelizer. All of the plots are interactive (generated using Plotly).

Consider the following spectrum for a wide-band signal, which has been decomposed into its four constituent channels, each containing a narrow-band signal.
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Say we wish to extract the four narrow band signals while lowering the sampling rate. There are many ways to do this. Let us first walk through the most intuitive way, which is a straightforward decimation (anti-alias-filtering followed by down-sampling).

<h3>Simple Decimation Filter</h3>
For illustration, let us extract the signal on channel 3 (denoted "Signal 3" in the previous plot). 

We first apply a digital frequency shift to center the desired channel around DC. In practice, this is done by multiplying the IQ samples in time with a complex sinusoid with the proper frequency.
<iframe src="https://paulxu.me/images/2025-08-25/shifted_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Next, we apply anti-aliasing filter to attenuate all other channels, preventing them from folding down to DC when we down-sample later. In this case, I used a 64-tap finite impulse response (FIR) low-pass filter synthesized using the **firwin** function in the **scipy.signals** module:

<iframe src="https://paulxu.me/images/2025-08-25/lpf_taps.html" width="800" height="450" frameborder="0"></iframe>

The filtered signal looks like this:
<iframe src="https://paulxu.me/images/2025-08-25/filtered_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Finally, we down-sample the filtered signal with a factor of 4.
<iframe src="https://paulxu.me/images/2025-08-25/decim_filtered_signal.html" width="800" height="450" frameborder="0"></iframe>

As an aside, checkout my post here on visualizing the down-sampling process in the frequency domain.

Notice that during decimation filtering, we are discarding a large amount of the samples, on which we have spent a considerable amount of computation resources. We had to convolve the wide-band input signal with a 64-tap FIR filter.

Furthermore, if we wish to extract the other three channels, we would need to quadruple the amount of computation. 


<h3>Polyphase Channelizer</h3>
A polyphase channelizer realizes the same effect as the decimation filter much more efficiently.

To illustrate this, let us consider the following wide-band waveform, consisting of four narrow-band signals, each with a perfectly flat spectrum:
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal_polyphase_input.html" width="800" height="450" frameborder="0"></iframe>

To aid visualization, let us also assume that the signal has a purely real spectrum. The intuition developed in this case can later be generalized to arbitrary waveforms with complex spectra. We can now plot the spectrum of the signal on a complex IQ plane:

<iframe src="https://paulxu.me/images/2025-08-25/complex_spectrum_wideband_input.html" width="850" height="500" frameborder="0"></iframe>

<h4> Polyphase Components of the Input Signal</h4>

First, the original signal is decomposed into four parallel streams, each operating at one-quarter of the original sampling rate. These streams are staggered with respect to one another by one-sample increments.

At first glance, it may seem surprising that the signals are down-sampled without any anti-aliasing filter. Intuitively, this should make it impossible to recover the desired signals on each channel, since the spectral content would fold over and overlap.

Indeed, if we examine only a single down-sampled stream, this is exactly what happens. However, when we consider all four streams together and decompose each into contributions from the four narrow-band signals, we find that each stream contains a unique mixture of these signals, distinguished by different phase shifts. The following plot illustrates this effect. Play with the slider to visualize the decomposition of different streams.

<iframe src="https://paulxu.me/images/2025-08-25/downsampled_streams_spectrum.html" width="850" height="650" frameborder="0"></iframe>
 
With each stream alone, it is indeed impossible to recover the four narrow-band signals, as expected from the Nyquist sampling criterion. However, the advantage of a polyphase channelizer is that by cleverly combining the four parallel streams (after some processing), it becomes possible to cancel out the unwanted signal components while preserving the one of interest. Let us see how this is possible, by looking at the frequency response of the filter.

<h4> Polyphase Components of the Filter</h4>
The filter we wish to apply to the input signal (h) has the following frequency response:
<iframe src="https://paulxu.me/images/2025-08-25/ideal_lpf_spectrum.html" width="850" height="500" frameborder="0"></iframe>

From the block diagram of the polyphase channelizer, we see that, just like the input signal, the filter itself is decomposed into 4 staggered, down-sampled sub-filters. These are called the polyphase components of the original filter h0

<iframe src="https://paulxu.me/images/2025-08-25/downsampled_filter_spectrum.html" width="850" height="500" frameborder="0"></iframe>


<h4> Polyphase Filter Bank </h4>
The following plot is slightly busy, but you can disable individual traces by clicking on the legend.

The first set of traces corresponds to the inputs to the polyphase filter bank. More specifically, each plot shows the contribution of a single channel (ch0, ch1, ch2, and ch3) to the four polyphase filters (h0, h1, h2, and h3). Recall from the block diagram that each filter receives contributions from all four channels, but with different delays.

The dotted lines represent the frequency responses of the polyphase filters (h0, h1, h2, and h3).

Finally, the last set of traces corresponds to the outputs of the polyphase filters, obtained by multiplying the input traces with their respective filter responses. In the time domain, this would be a convolution operation.

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_input.html" width="850" height="650" frameborder="0"></iframe>

To focus on the outputs of the filter bank, the following plot replicates the one above, except only the outputs are shown for clarity.

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output.html" width="850" height="650" frameborder="0"></iframe>

Lastly, after applying a one-sample delay to the outputs of h1, h2, and h3 and zero delay to the output of h0, we obtain the following signals. These are then summed to produce the final output of the polyphase channelizer.
<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output_delayed.html" width="850" height="650" frameborder="0"></iframe>

It should now be evident that the contributions from ch1, ch2, and ch3 cancel due to being out of phase, leaving only the contribution from ch0.



<!-- 
1. Each of $h_0$, $h_1$, $h_2$, $h_3$ contain 4 shifted copies of the original filter, folded on top of each other. As with $x_0$, $x_1$, $x_2$, $x_3$, the folded copies of the filter response have progressively faster phase rotations, due to the progressive delay in time.
2. $x_0_0$, $x_1_0$, $x_2_0$, $x_3_0$ pass through $h_0$, $h_3$, $h_2$, $h_1$ respectively. After combining the results, the only part that does not add up destructively is the part that would have remained after decimation filtering.-->

<h3>Sample Python Code to Test Polyphase Filter</h3>
The following Python code demonstrates the equivalence between a polyphase filter and the simple decimation filter.

```python
import numpy as np
from scipy.fft import fft, ifft

x = np.random.randn(12)
h = np.random.randn(6)

y_filt_down = ifft(fft(x,12)*fft(h,12))
y_filt_down = y_filt_down[0::3]

h0 = h[0::3]
h1 = h[1::3]
h2 = h[2::3]

x0 = x[0::3]
x1 = x[1::3]
x2 = x[2::3]

y0 = ifft(fft(x0,4)*fft(h0,4))
y1 = ifft(fft(x2,4)*fft(h1,4))
y2 = ifft(fft(x1,4)*fft(h2,4))

y_filt_poly = np.roll(y0, 0) + np.roll(y1, 1)  + np.roll(y2, 1)
```