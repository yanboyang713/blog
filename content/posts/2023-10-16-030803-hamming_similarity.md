---
title: "Hamming similarity"
draft: false
---

Hamming distance is the opposite of Hamming similarity Especially for the data of string.

To compute the Hamming similarity between two strings of equal length, you usually compute the Hamming distance first and then convert it to similarity. The Hamming distance between two strings of equal length is the number of positions at which the corresponding symbols are different.

```python
def hamming_distance(s1, s2):
    if len(s1) != len(s2):
        raise ValueError("Strings must be of the same length")
    return sum(c1 != c2 for c1, c2 in zip(s1, s2))
def hamming_similarity(s1, s2):
    distance = hamming_distance(s1, s2)
    return (len(s1) - distance) / len(s1)

s1 = "karolin"
s2 = "kathrin"

print(hamming_distance(s1, s2))
print(hamming_similarity(s1, s2))
```


## Reference List {#reference-list}

1.  [A Comprehensive Survey of Clustering Algorithms Xu, D. &amp; Tian, Y. Ann. Data. Sci. (2015)](https://link.springer.com/article/10.1007/s40745-015-0040-1)
