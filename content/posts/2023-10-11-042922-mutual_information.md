---
title: "Mutual information"
draft: false
---

To measure, based on information theory, how much information is shared by two clusters, between which the nonlinear correlation can be detected.

Mutual information (MI) is a measure used to gauge the amount of information shared between two random variables. In the context of clustering, it can be used to compare the true labels of data points with their assigned cluster labels to determine the quality of the clustering.

Given a set of true labels and a set of predicted labels from clustering, the mutual information between them can be calculated as:

\\[ MI(U, V) = \sum\_{i=1}^{n} \sum\_{j=1}^{m} P(U\_i, V\_j) \log \left( \frac{P(U\_i, V\_j)}{P(U\_i)P(V\_j)} \right) \\]

Where:

-   \\( U \\) and \\( V \\) are the sets of true and predicted labels, respectively.
-   \\( P(U\_i) \\) is the probability that a data point randomly selected from \\( U \\) has label \\( U\_i \\).
-   \\( P(V\_j) \\) is the probability that a data point randomly selected from \\( V \\) has label \\( V\_j \\).
-   \\( P(U\_i, V\_j) \\) is the joint probability that a data point has label \\( U\_i \\) in \\( U \\) and label \\( V\_j \\) in \\( V \\).

The higher the mutual information, the more the true labels and cluster labels agree with each other.

However, MI has a potential limitation: it doesn’t account for the chance agreement. This means even random labelings can have non-zero mutual information. To counter this, the normalized mutual information (NMI) is often used:

\\[ NMI(U, V) = \frac{2 \times MI(U, V)}{H(U) + H(V)} \\]

Where:

-   \\( H(U) \\) is the entropy of \\( U \\).
-   \\( H(V) \\) is the entropy of \\( V \\).

Entropy of a set \\( W \\) is given by:

\\[ H(W) = - \sum\_{i=1}^{n} P(W\_i) \log P(W\_i) \\]

NMI values range between 0 and 1, with 1 indicating perfect clustering and 0 indicating that the clustering is no better than random assignment.

In summary, mutual information can provide insights into the quality of clustering by measuring the similarity between true labels and cluster labels. Normalized mutual information adjusts for chance agreement, providing a more robust measure for comparing clustering results.


## [python]({{< relref "20230404161934-python.md" >}}) {#python--20230404161934-python-dot-md}

```python
import numpy as np
from sklearn.metrics import confusion_matrix

def mutual_information(U, V):
    # Compute the joint probability matrix
    joint_prob = confusion_matrix(U, V) / float(len(U))

    # Compute the marginal probabilities
    prob_U = np.sum(joint_prob, axis=1)
    prob_V = np.sum(joint_prob, axis=0)

    # Compute mutual information
    MI = 0.0
    for i in range(len(prob_U)):
        for j in range(len(prob_V)):
            if joint_prob[i][j] > 0:
                MI += joint_prob[i][j] * np.log2(joint_prob[i][j] / (prob_U[i] * prob_V[j]))
    return MI

def entropy(labels):
    prob = np.bincount(labels) / float(len(labels))
    return -np.sum(p * np.log2(p) for p in prob if p > 0)

def normalized_mutual_information(U, V):
    MI = mutual_information(U, V)
    entropy_U = entropy(U)
    entropy_V = entropy(V)
    return 2 * MI / (entropy_U + entropy_V)

# Example
true_labels = [0, 0, 1, 1, 2, 2]
cluster_labels = [0, 0, 2, 1, 2, 2]

print("MI:", mutual_information(true_labels, cluster_labels))
print("NMI:", normalized_mutual_information(true_labels, cluster_labels))

```


## Reference List {#reference-list}

1.  [A Comprehensive Survey of Clustering Algorithms Xu, D. &amp; Tian, Y. Ann. Data. Sci. (2015)](https://link.springer.com/article/10.1007/s40745-015-0040-1)
