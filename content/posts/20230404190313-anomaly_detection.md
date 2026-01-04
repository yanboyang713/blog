---
title: "Anomaly Detection"
draft: false
---

## Overview {#overview}

Anomaly detection is the task of identifying observations, events, or patterns that deviate from expected (i.e., **normal**) behavior. What counts as **normal** depends on the system, the time horizon, and assumptions about the data-generating process.


## Common Anomaly Types {#common-anomaly-types}

-   **Point anomaly**: a single observation is abnormal.
-   **Contextual anomaly**: abnormal only in a specific context (time, location, seasonality, operating mode).
-   **Collective anomaly**: a sequence/group is abnormal even if individual points appear normal.


## Common Approaches {#common-approaches}

-   **Supervised**: learn a classifier from labeled normal/anomalous examples (often highly imbalanced).
-   **Semi-supervised / one-class**: learn a model of normal behavior and flag deviations (e.g., one-class SVM, reconstruction models).
-   **Unsupervised**: detect outliers via distance/density/clustering assumptions (e.g., k-means variants, LOF, Isolation Forest).


## Modalities (Examples) {#modalities--examples}

-   **Networking**: flow-level statistics, packet traces, service metrics.
-   **Logs**: templates, sequences, and event-count features.
-   **Time series**: forecasting-based or reconstruction-based detectors (uni-/multivariate).


## Related Notes {#related-notes}

-   [Network anomaly detection]({{< relref "2023-11-08-001924-network_anomaly_detection.md" >}})
-   [Anomaly Detection by Using Streaming K-Means and Batch K-Means]({{< relref "2023-11-08-020753-anomaly_detection_by_using_streaming_k_means_and_batch_k_means.md" >}})
-   [Log message anomaly detection with oversampling]({{< relref "2023-11-30-214512-log_message_anomaly_detection_with_oversampling.md" >}})
-   [Multivariate Time Series Anomaly Detection]({{< relref "2024-07-10-055242-multivariate_time_series_anomaly_detection.md" >}})
-   [Deep Learning for multivariate time series data Anomaly Detection]({{< relref "20230103080850-deep_learning_for_multivariate_time_series_data_anomaly_detection.md" >}})
-   [Wireless Anomaly detection Project]({{< relref "20230216190214-anomaly_prediction.md" >}})
-   [networking autoencoders Un-Supervised Anomaly Detection]({{< relref "20230407040149-networking_autoencoders_un_supervised_anomaly_detection.md" >}})


## References {#references}

-   Chandola, V., Banerjee, A., &amp; Kumar, V. (2009). Anomaly detection: A survey. **ACM Computing Surveys**, 41(3), 1–58.
