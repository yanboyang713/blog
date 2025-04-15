---
title: "CPU cycles per packet (CPP)"
tags: ["CPP"]
draft: false
---

In order to measure the [Central Processing Unit (CPU)]({{< relref "2025-04-11-014334-cpu.md" >}}) cycles per packet (CPP) spent in each network stack component, we first use the Linux perf tool [37] to count the total CPU cycles consumed in a 60-second packet transmission (\\(Cycle\_{total}\\)), which is repeated 5 times. We also use perf to trace the function calls and measure the percentage of the overall CPU cycles spent in the corresponding function (\\(Cycle\_{percentage}\\)). With the total number of packets sent in a 60-second packet transmission (\\(N\_{packet}\\)), we can calculate the CPP of a specific function call as follows:
\\[CPP = \frac{Cycle\_{total}}{N\_{packet} \times Cycle\_{percentage}}\\]


## Reference List {#reference-list}

1.  Qi, S., Kulkarni, S. G., &amp; Ramakrishnan, K. K. (2020). Assessing container network interface plugins: Functionality, performance, and scalability. IEEE Transactions on Network and Service Management, 18(1), 656-671.
