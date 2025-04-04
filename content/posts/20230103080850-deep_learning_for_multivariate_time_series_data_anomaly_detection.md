---
title: "Deep Learning for multivariate time series data Anomaly Detection"
draft: false
---

[Anomaly detection]({{< relref "20230404190313-anomaly_detection.md" >}}) in multivariate [Time Series]({{< relref "20230404184340-time_series.md" >}}) data poses a particular challenge because it requires simultaneous consideration of temporal dependencies and relationships between variables. Recent deep learning-based works have made impressive progress in this field.

Time-series analysis refers to a range of tasks that aim to extract meaningful knowledge from time-ordered data; the extracted knowledge can be used not only to diagnose the past behavior but also to predict the future.

Widely-known examples of time-series analysis include classification, clustering, forecasting, and anomaly detection.

Anomaly detection, the process of identifying unexpected items or events from data.

Classical methods including linear model-based methods [3], distance-based methods [4], density-based methods [5], and support vector machines [6].

target systems become larger and more complex, those methods face limitations, namely an inability to manipulate multi-dimensional data or address a shortage of labeled anomalies.

categorize the types of anomalies related to time-series data as follows
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1672752229/screenshot/dhr0lgqfrjy4liyctmj9.png)

-   POINT ANOMALY
-   CONTEXTUAL ANOMALY
-   COLLECTIVE ANOMALY

After convolutional neural networks (CNN) revolutionized the field of computer vision [31], researchers also began to apply CNN to time-series data analysis [32]. CNN-based fault detection and diagnosis models showed their competence in handling multivariate time-series data captured from semiconductor manufacturing processes in [33]–[35].

Another method used CNN to recognize failures with converted images from vibration signals of pumps [37].

Wen et al. presented a time-series anomaly detection model using a CNN [62] that adopted a transfer-learning framework to resolve data sparsity issues.

On that matter, a recent CNN-based model was proposed [64] to detect gas leaks by monitoring flow noise inside the pipes.

Similarly, Tang et al. [17], taking advantage of the inter-pretability of visualized data, converted raw time-series data to images and split the continuous data into segments by windowing data without overlap. Afterwards, they fed the pre-processed data into a CNN-based classification model. Each segment was decomposed into the time domain and frequency domain with Fast Fourier Transform (FFT) and fused as an image by stacking time response image and frequency response image.

Bao et al. [16], imitating the recognition process of humans, transformed data as image files and fed them into stacked autoencoders (SAE) for anomaly classification


## cloud systems {#cloud-systems}

time-series anomaly detection on cloud systems with subsequent diagnosis of the current state and tracing of the root causes is important to maintain high service availability [66], [67].

CNN-based approaches [70], [71] have been verified to be effective on several datasets from global cloud enterprises.


## Networking {#networking}

[networking autoencoders Un-Supervised Anomaly Detection]({{< relref "20230407040149-networking_autoencoders_un_supervised_anomaly_detection.md" >}})

In addition, Zhao et al. [73] recently suggested a graph attention network-based method to detect anomalies in a big-data processing framework. They explicitly modeled correlations between sensors via attention layers, captured temporal dependence with GRU, and increased performance by jointly applying forecasting and reconstruction results.

LSTM


## Reference List {#reference-list}

1.  Choi, K., Yi, J., Park, C. and Yoon, S., 2021. Deep learning for anomaly detection in time-series data: review, analysis, and guidelines. IEEE Access.
