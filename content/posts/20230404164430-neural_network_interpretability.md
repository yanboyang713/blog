---
title: "neural network interpretability"
draft: false
---

[deep neural network]({{< relref "20230404164236-deep_neural_network.md" >}})

Along with the great success of deep neural networks, there is also growing concern about their black-box nature. The interpretability issue affects people’s trust on deep learning systems.

deep learning still has some significant disadvantages. DNNs are often found to exhibit unexpected behaviours.

To open the black-boxes of deep networks, many researchers started to focus on the model interpretability.

The definition of interpretability is the ability to explain or to present in understandable terms to a human [2].

One previous definition of interpretability is the ability to provide explanations in understandable terms to a human [18],

-   **Explanations**, ideally, should be logical decision rules(ifthen rules) or can be transformed to logical rules. However, people usually do not require explanations to be explicitly in a rule form (but only some key elements which can be used to construct explanations).
-   **Understandable terms** should be from the domain knowledge related to the task (or common knowledge according to the task).

There are three categorical of evaluation approaches for interpretability: application-grounded, human-grounded, and functionally-grounded [2].

-   Application-grounded evaluation involves conducting domain expert experiments within a real application.
-   Human-grounded evaluation is about conducting simpler human-subject experiments that maintain the essence of the target application. Such an evaluation is appealing when experiments with the target community is challenging. These evaluations can be completed with lay humans, allowing for both a bigger subject pool and less expenses, since we do not have to compensate highly trained domain experts.
-   Functionally-grounded evaluation requires no human experiments; instead, it uses some formal definition of interpretability as a proxy for explanation quality.

Although deep networks have shown great performance on some relatively large test sets, the real world environment is still much more complex. As some unexpected failures are inevitable, we need some means of making sure we are still in control.

There are **TWO** categorical and has two possible values, passive interpretation and active interpretability intervention.


## Reference List {#reference-list}

1.  <https://yzhang-gh.github.io/notes/ml/nn-interpretability.html#%E5%89%8D%E8%A8%80>
2.  Zhang, Y., Tiňo, P., Leonardis, A., &amp; Tang, K. (2021). A survey on neural network interpretability. IEEE Transactions on Emerging Topics in Computational Intelligence, 5(5), 726-742.
3.  Doshi-Velez, F., &amp; Kim, B. (2017). Towards a rigorous science of interpretable machine learning. arXiv preprint arXiv:1702.08608.
