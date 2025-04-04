---
title: "Davies–Bouldin indicator"
draft: false
---

Mainly for the data that has even density and distribution

The Davies–Bouldin index (DBI) is an indicator that measures the quality of clustering achieved by a clustering algorithm. Specifically, it provides a score that indicates the average similarity between each cluster and its most similar cluster, with the ideal score being the smallest possible value.

Here's a breakdown of the Davies–Bouldin index:

Within-cluster scatter: For each cluster, compute the average distance between each data point in the cluster and the centroid of the cluster. This provides a measure of the scatter or spread of the points within a cluster.

Of course! Here are the formulas for the Davies–Bouldin index written in LaTeX:

1.  ****Within-cluster scatter**** for each cluster:
    \\(S\_i = \frac{1}{|C\_i|} \sum\_{x \in C\_i} \left\\| x - c\_i \right\\|\\)
    Where:
    -   \\(C\_i\\) represents the i-th cluster.
    -   \\(c\_i\\) denotes the centroid of the i-th cluster.
    -   \\(|C\_i|\\) indicates the number of data points in the i-th cluster.

2.  ****Between-cluster separation**** for each pair of clusters:
    \\(d(c\_i, c\_j) = \left\\| c\_i - c\_j \right\\|\\)

3.  ****Similarity measure**** between two clusters:
    \\(R\_{ij} = \frac{S\_i + S\_j}{d(c\_i, c\_j)}\\)
    This formula computes the similarity between clusters \\(i\\) and \\(j\\).

4.  ****Davies–Bouldin Index****:
    \\(DBI = \frac{1}{N} \sum\_{i=1}^{N} \max\_{j \neq i} \left( R\_{ij} \right)\\)
    Where \\(N\\) represents the total number of clusters.

These formulas provide a comprehensive representation of the Davies–Bouldin index in mathematical terms.

In summary, the Davies–Bouldin index calculates the ratio of within-cluster scatter to between-cluster separation for each cluster, and then averages these ratios across all clusters. A lower DBI value indicates better clustering since it means that clusters are compact (small within-cluster scatter) and well-separated (large between-cluster separation).

When evaluating clustering results, it's important to consider multiple evaluation metrics alongside the DBI, as each has its strengths and limitations.


## [python]({{< relref "20230404161934-python.md" >}}) {#python--20230404161934-python-dot-md}

```python
import numpy as np
from sklearn.cluster import KMeans
from sklearn.metrics import pairwise_distances

def davies_bouldin_index(X, labels):
    n_cluster = len(np.bincount(labels))
    cluster_k = [X[labels == k] for k in range(n_cluster)]
    centroids = [np.mean(k, axis=0) for k in cluster_k]

    # Calculate the scatter (S_i) for each cluster
    S = [np.mean(pairwise_distances(cluster_k[i], [centroids[i]])) for i in range(n_cluster)]

    # Calculate the pairwise centroid distances (d(c_i, c_j))
    centroid_distances = pairwise_distances(centroids)
    np.fill_diagonal(centroid_distances, float('inf'))

    # Calculate the similarity (R_ij) between each pair of clusters
    R = np.zeros((n_cluster, n_cluster))
    for i in range(n_cluster):
        for j in range(n_cluster):
            if i != j:
                R[i, j] = (S[i] + S[j]) / centroid_distances[i, j]

    # Calculate the Davies–Bouldin index
    dbi = np.mean(np.max(R, axis=1))

    return dbi

# Sample dataset and clustering
X = np.array([[1, 2], [5, 8], [1.5, 1.8], [8, 8], [1, 0.6], [9, 11]])
kmeans = KMeans(n_clusters=2).fit(X)
labels = kmeans.labels_

# Calculate DBI
dbi_value = davies_bouldin_index(X, labels)
print(f"Davies–Bouldin index: {dbi_value}")
```
