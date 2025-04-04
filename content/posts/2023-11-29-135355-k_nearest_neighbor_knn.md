---
title: "K-Nearest Neighbor (KNN)"
draft: false
---

K-Nearest Neighbor (KNN) works by finding the nearest instances surrounding the
instance to be classified and predicting the output based on those K instances. It is a
non-parametric machine learning algorithm that does not make any assumptions about the
form of the mapping function [9]. To classify any given instance, it looks up its K neighbors
and assigns the instance to the appropriate class. The nearest neighbors are selected based
on a distance measure in the feature space. KNN is used by synthetic data generation
techniques such as SMOTE by creating new synthetic samples on the lines connecting the
existing minority sample to their k-nearest neighbors [10,11]. Since KNN plays a vital role
in the identification of misclassified instances in the case of BSMOTE, it was decided to
vary K and analyze the results.


## Reference List {#reference-list}

1.  Bagui, S. S., Mink, D., Bagui, S. C., &amp; Subramaniam, S. (2023). Determining Resampling Ratios Using BSMOTE and SVM-SMOTE for Identifying Rare Attacks in Imbalanced Cybersecurity Data. Computers, 12(10), 204.
