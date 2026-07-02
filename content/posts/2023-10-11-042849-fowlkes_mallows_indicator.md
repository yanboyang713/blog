---
title: "Fowlkes–Mallows indicator"
draft: false
---

The Fowlkes–Mallows index, often abbreviated as the FM index, is a measure used to evaluate the similarity between two clusterings (or partitions) of a set of data points. It is particularly useful in situations where the true clustering (or ground truth) is known, and you want to compare it against a proposed clustering.

Given two partitions, \\(U\\) and \\(V\\), of a set of \\(n\\) objects, the Fowlkes–Mallows index is defined in terms of pairs of objects:

1.  \\(TP\\) (True Positive): The number of pairs of objects that are in the same subset in \\(U\\) and in the same subset in \\(V\\).
2.  \\(FP\\) (False Positive): The number of pairs of objects that are in the same subset in \\(U\\) but in different subsets in \\(V\\).
3.  \\(FN\\) (False Negative): The number of pairs of objects that are in different subsets in \\(U\\) but in the same subset in \\(V\\).

The Fowlkes–Mallows index \\(B\\) is then defined as:
\\(B(U,V) = \frac{TP}{\sqrt{(TP + FP) \times (TP + FN)}}\\)

Properties:

-   The value of the Fowlkes–Mallows index ranges between 0 and 1. A value of 1 indicates that the two partitions being compared are identical, while a value of 0 indicates that the two partitions do not have any pairs in common.
-   It is symmetric, meaning that \\(B(U,V) = B(V,U)\\).
-   The FM index considers only pairs of objects that are clustered together in both partitions (or in neither partition). As such, it can be seen as a measure of the geometric mean of the precision and recall of the clustering.

Usage:
The Fowlkes–Mallows index can be used in various clustering contexts to assess the similarity between two partitions. It can be particularly helpful in evaluating clustering algorithms against ground truth partitions or comparing the performance of different clustering algorithms against each other.


## [python]({{< relref "20230404161934-python.md" >}}) {#python--20230404161934-python-dot-md}

```python
from itertools import combinations
from math import sqrt

def fowlkes_mallows_index(labels_true, labels_pred):
    """
    Compute the Fowlkes-Mallows index for two clusterings.

    :param labels_true: List of ground truth cluster labels
    :param labels_pred: List of predicted cluster labels
    :return: Fowlkes-Mallows index
    """
    if len(labels_true) != len(labels_pred):
        raise ValueError("Both lists must have the same length")

    # Compute pairs
    pairs_true = {(i, j) for i in range(len(labels_true)) for j in range(i+1, len(labels_true)) if labels_true[i] == labels_true[j]}
    pairs_pred = {(i, j) for i in range(len(labels_pred)) for j in range(i+1, len(labels_pred)) if labels_pred[i] == labels_pred[j]}

    TP = len(pairs_true & pairs_pred)
    FP = len(pairs_pred - pairs_true)
    FN = len(pairs_true - pairs_pred)

    if TP == 0:
        return 0.0
    else:
        return TP / sqrt((TP + FP) * (TP + FN))


# Example usage
labels_true = [0, 0, 1, 1, 2, 2]
labels_pred = [0, 0, 1, 2, 2, 2]
print(fowlkes_mallows_index(labels_true, labels_pred))  # This should print the FM index for the given clusterings

```


## Reference List {#reference-list}

1.  [A Comprehensive Survey of Clustering Algorithms Xu, D. &amp; Tian, Y. Ann. Data. Sci. (2015)](https://link.springer.com/article/10.1007/s40745-015-0040-1)
