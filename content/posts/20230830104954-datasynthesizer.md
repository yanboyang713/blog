---
title: "DataSynthesizer"
draft: false
---

DataSynthesizer, a tool that takes a sensitive dataset as input and generates a structurally and statistically similar [synthetic]({{< relref "20230830105258-synthetic_data.md" >}}) dataset with strong [privacy]({{< relref "20230516133528-privacy.md" >}}) guarantees.

data owner does not have to specify any parameters to start generating and sharing data safely and effectively.

DataSynthesizer consists of three high-level modules — DataDescriber, DataGenerator and ModelInspector.

-   DataDescriber, investigates the data types, correlations and distributions of the attributes in the private dataset, and produces a data summary, adding noise to the distributions to preserve privacy.
-   DataGenerator samples from the summary computed by DataDescriber and outputs synthetic data.
-   ModelInspector shows an intuitive description of the data summary that was computed by DataDescriber, allowing the data owner to evaluate the accuracy of the summarization process and adjust any parameters, if desired.

generate datasets that are structurally and statistically similar to the real data but that

1.  obviously synthetic to put the data owners at ease,
2.  offer strong privacy guarantees to prevent adversaries from extracting any sensitive information.

DataSynthesizer can operate in one of three modes:

-   In correlated attribute mode, we learn a differentially private Bayesian network capturing the correlation structure between attributes, then draw samples from this model to construct the result dataset.
-   In cases where the correlated attribute mode is too computationally expensive or when there is insuffcient data to derive a reasonable model, one can use independent attribute mode. In this mode, a histogram is derived for each attribute, noise is added to the histogram to achieve differential privacy, and then samples are drawn for each attribute.
-   Finally, for cases of extremely sensitive data, one can use random mode that simply generates type-consistent random values for each attribute.


## Reference List {#reference-list}

1.  Ping, H., Stoyanovich, J., &amp; Howe, B. (2017, June). Datasynthesizer: Privacy-preserving synthetic datasets. In Proceedings of the 29th International Conference on Scientific and Statistical Database Management (pp. 1-5).
