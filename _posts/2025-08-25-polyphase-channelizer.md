---
title: "Polyphase Channelizer"
date: 2025-08-25
tags: dsp notes study communication
---

This post explores the intuition behind the polyphase channelizer. All of the plots are interactive. 

The polyphase channelizer is a powerful tool used to efficiently split a wideband signal into multiple narrower channels. The plot below shows a simplified block diagram illustrating how a single channel is extracted, which we will examine first. Once we understand this process, we will modify the block diagram to extract all channels simultaneously.

<iframe src="https://paulxu.me/images/2025-08-25/block_diagram.png" width="1000" frameborder="0"></iframe>

Before introducing the polyphase channelizer, let’s first look at the straightforward method of extracting a channel using decimation (applying an anti-alias filter followed by down-sampling), and examine its limitations.

<h3>Decimation Filter</h3>
Consider the following wide-band input signal \\(x\\). I have plotted its spectrum, along with the spectra for its four constituent channels (ch0, ch1, ch2, and ch3).
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

For illustration, let us extract the signal on channel 2 (denoted "ch2"). 

We first apply a digital frequency shift to center the desired channel around DC (0 Hz). In practice, this is done by multiplying the IQ samples in time with a complex sinusoid with the proper frequency.
<iframe src="https://paulxu.me/images/2025-08-25/shifted_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>

Next, we apply a anti-aliasing filter to attenuate all other channels, preventing them from folding down to DC when we down-sample later. In this case, I used a 64-tap finite impulse response (FIR) low-pass filter synthesized using the **firwin** function in the **scipy.signals** module:

<iframe src="https://paulxu.me/images/2025-08-25/lpf_taps.html" width="800" height="450" frameborder="0"></iframe>

The filtered signal looks like this:
<iframe src="https://paulxu.me/images/2025-08-25/filtered_wideband_signal.html" width="800" height="450" frameborder="0"></iframe>
The contents in ch0, ch1 and ch3 are heavily attenuated.

Finally, we down-sample the filtered signal with a factor of 4, since the high sample rate is no longer needed to fully capture the information in the desired channel.
<iframe src="https://paulxu.me/images/2025-08-25/decim_filtered_signal.html" width="800" height="450" frameborder="0"></iframe>

We can see that the extracted signal almost perfectly matches the desired signal, due the application of the anti-aliasing filter.

> As an aside, checkout my post here on visualizing the down-sampling process in the frequency domain.

Notice that in the decimation filtering step, we discard a large fraction of the samples (three out of four in this example), even though we spent significant computation to produce them by convolving the wide-band input signal with a 64-tap FIR filter.

If we then wanted to extract the other three channels as well, the computational cost would be roughly four times higher; this is extremely inefficient.

<h3>Polyphase Channelizer</h3>
A polyphase channelizer achieves the same result as a decimation filter, but with much greater efficiency.

To illustrate its working principle, let us consider the following wide-band waveform, consisting of four narrow-band signals, each with a perfectly flat spectrum:
<iframe src="https://paulxu.me/images/2025-08-25/wideband_signal_polyphase_input.html" width="850" height="500" frameborder="0"></iframe>

To aid visualization, let us also assume that the signal has a purely real spectrum. The intuition developed in this case can later be generalized to arbitrary waveforms with complex spectra. We can now plot the spectrum of the signal on a complex IQ plane:

<iframe src="https://paulxu.me/images/2025-08-25/complex_spectrum_wideband_input.html" width="850" height="550" frameborder="0"></iframe>

<h4> Polyphase Decomposition of the Input Signal</h4>

Following the block diagram for the polyphase channelizer, the original signal \\(x\\) is decomposed into four parallel streams \\(x_0\\), \\(x_1\\), \\(x_2\\), \\(x_3\\), staggered with respect to one another by one-sample increments and down-sampled by a factor of 4.

At first glance, it may seem surprising that the signals are down-sampled without any anti-aliasing filter. Intuitively, this should make it impossible to recover the desired signals on each channel, since the spectral content would fold over and overlap.

Indeed, if we examine only a single down-sampled stream, this is exactly what happens. However, when we consider all four streams together and decompose each into contributions from the four narrow-band signals, we find that each stream contains a unique mixture of these signals, distinguished by different phase shifts. The following plot illustrates this effect. Play with the slider to visualize the decomposition of different streams.

<iframe src="https://paulxu.me/images/2025-08-25/downsampled_streams_spectrum.html" width="850" height="650" frameborder="0"></iframe>
 
With each stream alone, it is indeed impossible to recover the four narrow-band signals, as expected from the Nyquist sampling criterion. However, the advantage of a polyphase channelizer is that by cleverly combining the four parallel streams (after some processing), it becomes possible to cancel out the unwanted signal components while preserving the one of interest. Let us see how this is done, by looking at the frequency response of the filter.

<h4> Polyphase Components of the Filter</h4>
The filter \\(h\\) we wish to apply to the input signal has the following frequency response:
<iframe src="https://paulxu.me/images/2025-08-25/ideal_lpf_spectrum.html" width="850" height="500" frameborder="0"></iframe>

From the block diagram of the polyphase channelizer, we see that instead of applying \\(h\\) directly to the signals \\(x_0\\), \\(x_1\\), \\(x_2\\), and \\(x_3\\), we instead apply a four different filters \\(h_0\\), \\(h_1\\), \\(h_2\\), and \\(h_3\\). Without delving into the mathematical derivation, I will just note that these filters are derived from \\(h\\) the same way that \\(x_0\\), \\(x_1\\), \\(x_2\\), and \\(x_3\\) are derived from \\(x\\): they are staggered down-sampled versions of \\(h\\). These smaller filters are called the polyphase components of the original filter \\(h\\).

From the complex IQ plot of the polyphase filter responses, we see that they all have the same magnitude response. However, due to the different applied delay, they carry different phase shifts in the frequency domain.
<iframe src="https://paulxu.me/images/2025-08-25/downsampled_filter_spectrum.html" width="850" height="650" frameborder="0"></iframe>

How do the polyphase filters \\(h_0\\), \\(h_1\\), \\(h_2\\), and \\(h_3\\) interact with the down-sampled, aliased signals \\(x_0\\), \\(x_1\\), \\(x_2\\), and \\(x_3\\)? The plot below illustrates this relationship. It may look a bit busy at first, but you can simplify the view by enabling or disabling individual traces through the legend.

The first set of traces corresponds to the inputs to the polyphase filter bank. More specifically, each plot shows the contribution of a single channel (ch0, ch1, ch2, and ch3). Recall that each of \\(x_0\\), \\(x_1\\), \\(x_2\\), and \\(x_3\\) contains a unique mixture of the four channels.

The dotted lines represent the frequency responses of the polyphase filters (h0, h1, h2, and h3).

Finally, the last set of traces corresponds to the outputs of the polyphase filters, obtained by multiplying the input traces with their respective filter responses. In the time domain, this would be a convolution operation.

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_input.html" width="850" height="650" frameborder="0"></iframe>

To focus on the outputs of the filter bank, the following plot replicates the one above, except only the outputs are shown for clarity.

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output.html" width="850" height="650" frameborder="0"></iframe>

At this point, it is still not clear how combining these signals together would allow us to extract the content in one channel, without it being corrupted by the aliased contributions from the other channels. The answer is in the final stage of the block diagram, where we apply a one-sample delay to the outputs of \\(h_1\\), \\(h_2\\), \\(h_3\\), and zero delay to the output of \\(h_0\\). The resulting spectra of this operation is shown below.
<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output_delayed.html" width="850" height="650" frameborder="0"></iframe>

It is now evident that the contributions from ch1, ch2, and ch3 cancel due to being out of phase, leaving only the contribution from ch0.

<h4> Extraction of Other Channels</h4>
A closer examination of the previous figure gives us some insight into how the other channels, besides ch0, can be extracted. By applying an appropriate phase shift to the outputs of the polyphase filter bank, we can cause any of ch1, ch2, or ch3 to sum constructively, while the remaining channels cancel out due to being out of phase. 

> Recall that a phase shift of \(\phi\) corresponds to a rotation of \(\phi\) degrees of the frequency spectrum for all positive frequencies, and a rotation of -\(\phi\) degrees for all negative frequencies. Checkout my post on this topic here.

With that in mind, we can now introduce the complete block diagram for the polyphase channelizer:

<iframe src="https://paulxu.me/images/2025-08-25/block_diagram_complete.png" width="1000" frameborder="0"></iframe>
Here, a mulplication by \\(j\\) corresponds to a phase shift of 90 degrees, while a multiplication by \\(-1\\) corresponds to a phase shift of 180 degrees.

Earlier, we saw that at the first output y0, only ch0 adds in phase, while ch1, ch2, and ch3 cancel out. Let’s now examine the other three outputs, each of which incorporates additional phase shifts.

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output_delayed_ch1.html" width="850" height="650" frameborder="0"></iframe>

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output_delayed_ch2.html" width="850" height="650" frameborder="0"></iframe>

<iframe src="https://paulxu.me/images/2025-08-25/filter_bank_output_delayed_ch3.html" width="850" height="650" frameborder="0"></iframe>

We can see that at each of the outputs y1, y2, and y3, only ch1, ch2, and ch3 respectively add in phase.

<h4> Final Implementation </h4>
A careful reader may notice that the combining matrix at the output of the polyphase filter bank looks very familiar. In fact, it is identical to the matrix multiplication implementation of the discrete Fourier transform (DFT). This observation allows us to simplify the block diagram of the polyphase channelizer as follows:   

<iframe src="https://paulxu.me/images/2025-08-25/block_diagram_dft.png" width="1000" frameborder="0"></iframe>

Compared to the straightforward decimation filter, the polyphase channel extractor is far more efficient. Each FIR filter in the polyphase structure uses only one quarter of the taps of the original filter, significantly reducing the computational load. Moreover, instead of extracting one channel at a time, the polyphase method allows us to extract all four channels simultaneously (with only a small amount of additional overhead). This parallelism makes polyphase channelizers particularly attractive in systems where multiple channels must be processed in real time.

<h4> Non-ideal FIR Filters </h4>
Up to this point, we have only considered low-pass filters with ideal brick-wall responses, where the gain is exactly zero outside the filter bandwidth. In practice, however, FIR filters cannot achieve such a response and will exhibit some ripples outside the cutoff frequency.

This raises an important question: what happens when we down-sample a real FIR filter which does not have zero gain outside the passband? 
For instance, consider the following non-ideal filter:

<iframe src="https://paulxu.me/images/2025-08-25/nonideal_filter_freq.html" width="850" height="500" frameborder="0"></iframe>
As long as the signal completely fits inside the filter bandwidth, the it will pass through the filter without any distortion.

But what happens when we try to implement this filter using a polyphase filter bank? Decomposing this filter into its four polyphase components reveals that each component is distorted due to aliasing, since down-sampling folds the out-of-band filter response back into the passband of each polyphase branch:

<iframe src="https://paulxu.me/images/2025-08-25/nonideal_filter_downsamp.html" width="850" height="500" frameborder="0"></iframe>

If all of the polyphase components are distorted, how can a narrowband signal still pass through the filter bank without distortion? The answer lies in the magic of the polyphase structure: the phase shifts introduced by the delays in the downsampled input and those from the polyphase filter components interact in just the right way. As a result, the outputs \\(y_0\\), \\(y_1\\), \\(y_2\\), and \\(y_3\\) acquire the necessary phases such that when combined, the distortions cancel out, leaving us with the expected output \\(y\\).

<iframe src="https://paulxu.me/images/2025-08-25/nonideal_filter_output.html" width="850" height="500" frameborder="0"></iframe>


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