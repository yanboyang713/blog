---
title: "The difference between parameter and hyperparameter"
draft: false
---

-   Model parameters: These are the parameters that are estimated by the model from the given data. For example the weights of a deep neural network.
-   Model hyperparameters: These are the parameters that cannot be estimated by the model from the given data. These parameters are used to estimate the model parameters. For example, the learning rate in deep neural networks.

| PARAMETERS                                                                                      | HYPERPARAMETERS                                                                                                                                                                                      |
|-------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| They are required for making predictions                                                        | They are required for estimating the model parameters                                                                                                                                                |
| They are estimated by optimization algorithms(Gradient Descent, Adam, Adagrad)                  | They are estimated by hyperparameter tuning                                                                                                                                                          |
| They are not set manually                                                                       | They are set manually                                                                                                                                                                                |
| The final parameters found after training will decide how the model will perform on unseen data | The choice of hyperparameters decide how efficient the training is. In gradient descent the learning rate decide how efficient and accurate the optimization process is in estimating the parameters |


## Reference List {#reference-list}

1.  <https://machinelearningmastery.com/difference-between-a-parameter-and-a-hyperparameter/>
2.  <https://neptune.ai/blog/hyperparameter-tuning-in-python-complete-guide>
3.  <https://www.geeksforgeeks.org/difference-between-model-parameters-vs-hyperparameters/>
