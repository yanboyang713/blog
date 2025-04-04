---
title: "dimension reduction"
draft: false
---

## Types {#types}

Dataset size reduction can be performed in one of the two ways [5]

-   feature set reduction ([feature selection]({{< relref "2023-11-22-110032-feature_selection.md" >}}))
-   sample set reduction.

Traditional dimensionality reduction approaches:

-   [Principal Component Analysis (PCA)]({{< relref "20230506180826-principal_component_analysis_pca.md" >}})
-   [linear discriminant analysis (LDA)]({{< relref "2023-10-16-054536-linear_discriminant_analysis_lda.md" >}})

the dimensionality of data increases, the computational cost of traditional dimensionality reduction methods grows exponentially, and the computation becomes prohibitively intractable.

These drawbacks have triggered the development

-   [random projection]({{< relref "2023-10-16-052501-random_projection.md" >}}) reduced time cost

However, the RP transformation matrix is generated without considering the intrinsic structure of the original data and usually leads to relatively high distortion.


## Reference List {#reference-list}

1.  <https://zhuanlan.zhihu.com/p/159285110?utm_id=0>
2.  <https://zhuanlan.zhihu.com/p/62470700?utm_id=0>
3.  Xie, H., Li, J., &amp; Xue, H. (2017). A survey of dimensionality reduction techniques based on random projection. arXiv preprint arXiv:1706.04371.
4.  Boutsidis, C., Zouzias, A., &amp; Drineas, P. (2010). Random projections for $ k $-means clustering. Advances in neural information processing systems, 23.
5.  Jović, A., Brkić, K., &amp; Bogunović, N. (2015, May). A review of feature selection methods with applications. In 2015 38th international convention on information and communication technology, electronics and microelectronics (MIPRO) (pp. 1200-1205). Ieee.
