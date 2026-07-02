---
title: "K-means clustering algorithm"
draft: false
---

K-means clustering is the most commonly used clustering algorithm. It's a [Centroid-based Clustering]({{< relref "2023-09-29-234401-centroid_based_clustering.md" >}}) algorithm and the simplest unsupervised learning algorithm.

This algorithm tries to minimize the variance of data points within a cluster. It's also how most people are introduced to unsupervised machine learning.

K-means is best used on smaller data sets because it iterates over all of the data points. That means it'll take more time to classify data points if there are a large amount of them in the data set.

Since this is how k-means clusters data points, it doesn't scale well.


## Implementation {#implementation}

Using numpy array:

```python
import numpy as np
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# Example data
data = np.array([[2, 3], [4, 6], [8, 8], [3, 2], [10, 11], [15, 13]])

# Number of clusters
k = 2

# Create and fit KMeans model
kmeans = KMeans(n_clusters=k)
kmeans.fit(data)

# Get cluster labels and centers
labels = kmeans.labels_
cluster_centers = kmeans.cluster_centers_

# Visualize the clusters
plt.scatter(data[:, 0], data[:, 1], c=labels, cmap='viridis')
plt.scatter(cluster_centers[:, 0], cluster_centers[:, 1], c='red', marker='x', s=200)
plt.show()
```

```python
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# Example data as a NumPy array
data = np.array([[2, 3], [4, 6], [8, 8], [3, 2], [10, 11], [15, 13]])

# Convert the data to a Pandas DataFrame
df = pd.DataFrame(data, columns=['X', 'Y'])

# Number of clusters
k = 2

# Create and fit KMeans model
kmeans = KMeans(n_clusters=k)
kmeans.fit(df)

# Get cluster labels and centers
labels = kmeans.labels_
cluster_centers = kmeans.cluster_centers_

# Visualize the clusters
plt.scatter(df['X'], df['Y'], c=labels, cmap='viridis')
plt.scatter(cluster_centers[:, 0], cluster_centers[:, 1], c='red', marker='x', s=200)
plt.show()
```


## optimal number of clusters {#optimal-number-of-clusters}


### Elbow Method {#elbow-method}

```python
  from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

distortions = []
K = range(1, 10)
for k in K:
    kmeanModel = KMeans(n_clusters=k)
    # if use Pandas DataFrame, change to df
    kmeanModel.fit(data)
    distortions.append(kmeanModel.inertia_)

plt.plot(K, distortions, 'bx-')
plt.xlabel('Number of Clusters')
plt.ylabel('Distortion')
plt.title('Elbow Method')
plt.show()

```


## Reference List {#reference-list}

1.  <https://www.freecodecamp.org/news/8-clustering-algorithms-in-machine-learning-that-all-data-scientists-should-know/>
