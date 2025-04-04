---
title: "Signal to Noise Ratio (SNR)"
draft: false
---

Signal to Noise Ratio (SNR)} is calculated using the formula \\(SNR = \frac{P\_{S}}{P\_{I+N}}\\), where \\(P\_{S}\\) represents the power of the signal and \\(P\_{I+N}\\) represents the combined power of the interference and background noise.

It is directly connected with the process of packet capture by a receiver. If a receiver captures a packet, two conditions must be met:

1.  The effective signal strength must be greater than the receiver sensitivity
2.  The actual SNR at receiver must be greater than the SNR threshold of the receiver

Therefore, we can estimate the packet reception rate based on SNR. In practice, we should avoid choosing links with SNR falling in the bad or the transitional region.


## Reference List {#reference-list}

1.  Yuan, D., Kanhere, S. S., &amp; Hollick, M. (2017). Instrumenting Wireless Sensor Networks—A survey on the metrics that matter. Pervasive and Mobile Computing, 37, 45-62.
