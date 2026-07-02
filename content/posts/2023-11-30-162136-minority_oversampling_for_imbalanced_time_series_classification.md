---
title: "Minority oversampling for imbalanced time series classification"
draft: false
---

[Time Series]({{< relref "20230404184340-time_series.md" >}})

Many vital real-world applications involve time-series data with skewed distribution. Compared to traditional imbalanced learning problems, the classification of imbalanced time-series data is more challenging due to the high dimensionality and high inter-variable correlation.

structure-preserving Oversampling method to resolve the High-dimensional Imbalanced Timeseries classification (OHIT).
OHIT leverages a density-ratio-based shared nearest neighbor clustering algorithm to capture the modes of minority class in high-dimensional space.
It for each mode applies the shrinkage technique of large-dimensional covariance matrix to obtain an accurate and reliable covariance structure.

The structure-preserving synthetic samples are eventually generated based on the multivariate Gaussian distribution with the estimated covariance matrix. In addition, to further promote the performance of classifying imbalanced time-series data, we integrate OHIT into boosting framework to obtain a new ensemble algorithm OHITBoost.
Extensive experiments on several publicly available time-series datasets (including unimodal and multimodal) demonstrate the effectiveness of OHIT and OHITBoost in terms of F1, G-mean, and AUC.


## Reference List {#reference-list}

1.  Zhu, T., Luo, C., Zhang, Z., Li, J., Ren, S., &amp; Zeng, Y. (2022). Minority oversampling for imbalanced time series classification. Knowledge-Based Systems, 247, 108764.
