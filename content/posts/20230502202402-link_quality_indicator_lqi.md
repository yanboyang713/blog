---
title: "Link Quality Indicator (LQI)"
draft: false
---

Link Quality Indicator (LQI) is used to assess the strength and quality of a received packet. The value of LQI is reported as an integer that should range from 0 to 255. A higher LQI value generally indicates a stronger and more reliable link, while a lower value suggests a weaker or less reliable link.

Link Quality Indicator (LQI) is used to assess the strength and quality of a received packet. The value of LQI is reported as an integer that should range from 0 to 255. but the implementation is vendor-specific.

There are different opinions on whether RSSI or LQI should be used to represent link quality.

It is advocated in [64] that LQI should be preferred as it correlates better with PRR than RSSI. A different opinion is held in [61]. We argue that RSSI and LQI provide different characterizations of signal quality. RSSI measures the power of the signal while LQI measures the level of distortion of the signal. Therefore, the combination of both should give better link quality estimation than any of them alone.


## Reference List {#reference-list}

1.  Yuan, D., Kanhere, S. S., &amp; Hollick, M. (2017). Instrumenting Wireless Sensor Networks—A survey on the metrics that matter. Pervasive and Mobile Computing, 37, 45-62.
