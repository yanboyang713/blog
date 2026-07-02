---
title: "Silhouette coefficient"
draft: false
---

The Silhouette Coefficient (often denoted as \\( s \\)) is a metric used to calculate the goodness of a clustering algorithm. Its value ranges from -1 to 1, where a high value indicates that the object is well matched to its own cluster and poorly matched to neighboring clusters. If most objects have a high value, then the clustering configuration is appropriate. If many points have a low or negative value, then the clustering configuration may have too many or too few clusters.

The Silhouette Coefficient is defined for each sample and is composed of two scores:

1.  \\(a\\): The average distance between the sample and all other points in the same cluster.
2.  \\(b\\): The smallest average distance between the sample and all points in any other cluster, of which the sample is not a part.

The Silhouette Coefficient \\(s\\) for a single sample is defined as:

\\(s = \frac{b - a}{\max(a, b)}\\)

For clarity:

-   \\(s\\) is close to 1 implies the sample is well clustered.
-   \\(s\\) close to 0 implies the sample is on or very close to the decision boundary between two neighboring clusters.
-   \\(s\\) close to -1 implies the samples might have been assigned to the wrong cluster.

To find the overall Silhouette Coefficient of the data, one typically takes the average of the silhouette score for each instance.

This metric can be particularly useful because:

1.  It provides a succinct score for each sample regarding its appropriateness to its assigned cluster.
2.  It doesn't require the true labels of the samples — meaning it can be used on unsupervised datasets.

However, remember that, like any metric, the Silhouette Coefficient has its limitations and is just one of many metrics that can be used to evaluate the quality of a clustering algorithm. It's also worth noting that a high silhouette score doesn't necessarily mean the clustering is correct, especially if the data doesn't inherently form clusters.


## [python]({{< relref "20230404161934-python.md" >}}) {#python--20230404161934-python-dot-md}

```python
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score, silhouette_samples

# Generate some sample data
X, _ = make_blobs(n_samples=500, centers=4, random_state=42, cluster_std=1.0)

# Apply KMeans clustering
n_clusters = 4
kmeans = KMeans(n_clusters=n_clusters, random_state=42)
cluster_labels = kmeans.fit_predict(X)

# Calculate the overall silhouette score
silhouette_avg = silhouette_score(X, cluster_labels)
print(f"For n_clusters = {n_clusters}, the average silhouette_score is : {silhouette_avg}")

# Calculate the silhouette score for each sample
sample_silhouette_values = silhouette_samples(X, cluster_labels)
```
