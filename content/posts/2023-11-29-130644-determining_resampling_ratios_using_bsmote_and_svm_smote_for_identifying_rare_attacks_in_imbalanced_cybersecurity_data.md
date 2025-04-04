---
title: "Determining Resampling Ratios Using BSMOTE and SVM-SMOTE for Identifying Rare Attacks in Imbalanced Cybersecurity Data"
draft: false
---

## Background {#background}

-   [resampling]({{< relref "2023-11-29-131950-resampling.md" >}})


## Why {#why}

The ratio of actual attacks to benign data is significantly high and as such forms highly imbalanced datasets.
though network attacks are increasing steadily, the ratio of attacks to regular network traffic is still significantly less, creating highly imbalanced datasets.
And this leads to the problem of effectively training ML models to detect and classify malicious traffic, especially rare malicious (attacks) traffic. Hence, predicting rare attacks in imbalanced datasets has become a significant problem.


## How {#how}

In this work, we address this issue using data resampling techniques. Though there are several oversampling and undersampling techniques available, how these oversampling and undersampling techniques are most effectively used is addressed in this paper.


## Methods {#methods}

-   Two oversampling techniques, Borderline SMOTE and SVM-SMOTE, are used for oversampling minority data
    Both the oversampling techniques use KNN after selecting a random minority sample

point, hence the impact of varying KNN values on the performance of the oversampling technique is
also analyzed

-   random undersampling is used for undersampling majority data.
-   Random Forest is used for classification of the rare attacks.

[Network Intrusion Detection Systems (NIDSs)]({{< relref "2023-11-08-034959-network_intrusion_detection_systems_nidss.md" >}}) were comprised of either Signature Based Detection or Behavior Based Detection. The former relied on the signature of the attack to be similar to the known signatures while the later compared the profile of the attack and compared it with the normal behavior of the standard profiles [3].


## Reference List {#reference-list}

1.  Bagui, S. S., Mink, D., Bagui, S. C., &amp; Subramaniam, S. (2023). Determining Resampling Ratios Using BSMOTE and SVM-SMOTE for Identifying Rare Attacks in Imbalanced Cybersecurity Data. Computers, 12(10), 204.
