---
title: "Clustering Algorithms Evaluation Indicators"
draft: false
---

The main purpose of evaluation indicator is to test the validity of [Clustering Algorithms]({{< relref "2023-09-29-233551-clustering_algorithms.md" >}}).

Evaluation indicators can be divided into two categories: the internal evaluation indicators and the external evaluation indicators.


### The internal evaluation indicators {#the-internal-evaluation-indicators}

It can’t absolutely judge which algorithm is better when the scores of two algorithms are not equal based on the internal evaluation indicators.

There are three commonly used internal indicators.

-   [Davies–Bouldin indicator]({{< relref "2023-10-16-034117-davies_bouldin_indicator.md" >}}) the data that has even density and distribution
-   [Dunn indicator]({{< relref "2023-10-16-035722-dunn_indicator.md" >}}) requires computing distances between all pairs of clusters and within all clusters. This can be a limiting factor for very large datasets.
-   [Silhouette coefficient]({{< relref "2023-10-16-040835-silhouette_coefficient.md" >}})


### the external evaluation indicators {#the-external-evaluation-indicators}

The external evaluation, which is called as the gold standard for testing method, takes the external data to test the validity of algorithm.

-   [Jaccard indicator ]({{< relref "2023-10-11-042806-jaccard_indicator.md" >}})is a measure of similarity between two sets
-   [Hamming similarity]({{< relref "2023-10-16-030803-hamming_similarity.md" >}}) for the data of string
-   [Rand indicator]({{< relref "2023-10-11-042609-rand_indicator.md" >}})
-   [F indicator]({{< relref "2023-10-11-042724-f_indicator.md" >}})
-   [Fowlkes–Mallows indicator]({{< relref "2023-10-11-042849-fowlkes_mallows_indicator.md" >}})
-   [Mutual information]({{< relref "2023-10-11-042922-mutual_information.md" >}})
-   [Confusion matrix]({{< relref "2023-10-11-043001-confusion_matrix.md" >}})


## Reference List {#reference-list}

1.  [A Comprehensive Survey of Clustering Algorithms Xu, D. &amp; Tian, Y. Ann. Data. Sci. (2015)](https://link.springer.com/article/10.1007/s40745-015-0040-1)
