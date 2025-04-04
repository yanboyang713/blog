---
title: "statistical relational learning (SRL)"
draft: false
---

Statistical Relational Learning (SRL) is a subdiscipline of artificial intelligence and [machine learning]({{< relref "20230616191354-machine_learning.md" >}}) that is concerned with domain models that exhibit both uncertainty (which can be dealt with using statistical methods) and complex, relational structure.

Note that SRL is sometimes called Relational Machine Learning (RML) in the literature.

Typically, the [Knowledge representation]({{< relref "20230828122632-knowledge_representation.md" >}}) formalisms developed in SRL use (a subset of) [first-order logic]({{< relref "2023-10-30-031550-first_order_logic.md" >}}) to describe relational properties of a domain in a general manner (universal quantification) and draw upon probabilistic graphical models (such as Bayesian networks or Markov networks) to model the uncertainty; some also build upon the methods of inductive logic programming. Significant contributions to the field have been made since the late 1990s.

As is evident from the characterization above, the field is not strictly limited to learning aspects; it is equally concerned with [reasoning]({{< relref "20230828122745-reasoning.md" >}}) (specifically probabilistic inference) and knowledge representation. Therefore, alternative terms that reflect the main foci of the field include statistical relational learning and reasoning (emphasizing the importance of reasoning) and first-order probabilistic languages (emphasizing the key properties of the languages with which models are represented).

SRL systems typically support the learning of the SRL model from data, and support the ability to answer queries using [probabilistic inference]({{< relref "2023-10-30-212339-probabilistic_inference.md" >}}).


## Canonical tasks {#canonical-tasks}

-   link-based [clustering]({{< relref "2023-09-29-233551-clustering_algorithms.md" >}}), i.e. the grouping of similar objects, where similarity is determined according to the links of an object, and the related task of collaborative filtering, i.e. the filtering for information that is relevant to an entity (where a piece of information is considered relevant to an entity if it is known to be relevant to a similar entity)

-   [Markov Logic Networks (MLN)]({{< relref "2023-11-06-023113-markov_logic_networks_mln.md" >}}) [4]


## Book {#book}

-   Introduction to Statistical Relational Learning (Adaptive Computation and Maching Learning) by Lise Getoor (Editor)


## Reference List {#reference-list}

1.  <https://blog.csdn.net/PolarisRisingWar/article/details/127395263>
2.  <https://en.wikipedia.org/wiki/Statistical_relational_learning>
3.  Getoor, L., &amp; Mihalkova, L. (2011, June). Learning statistical models from relational data. In Proceedings of the 2011 ACM SIGMOD International Conference on Management of data (pp. 1195-1198).
4.  Khosravi, H., &amp; Bina, B. (2010). A survey on statistical relational learning. In Advances in Artificial Intelligence: 23rd Canadian Conference on Artificial Intelligence, Canadian AI 2010, Ottawa, Canada, May 31–June 2, 2010. Proceedings 23 (pp. 256-268). Springer Berlin Heidelberg.
