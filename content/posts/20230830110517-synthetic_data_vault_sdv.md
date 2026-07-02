---
title: "Synthetic Data Vault (SDV)"
draft: false
---

Synthetic Data Vault (SDV), a system that builds generative models of [relational database]({{< relref "20230830125058-relational_database.md" >}}). We are able to sample from the model and create synthetic data, hence the name SDV. When implementing the SDV, we also developed an algorithm that computes statistics at the intersection of related database tables. We then used a stateof-the-art multivariate modeling approach to model this data. The SDV iterates through all possible relations, ultimately creating a model for the entire database. Once this model is computed, the same relational information allows the SDV to synthesize data by sampling from any part of the database.


## Reference List {#reference-list}

1.  Patki, N., Wedge, R., &amp; Veeramachaneni, K. (2016, October). The synthetic data vault. In 2016 IEEE International Conference on Data Science and Advanced Analytics (DSAA) (pp. 399-410). IEEE.
