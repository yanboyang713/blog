---
title: "hyperparameter tuning"
draft: false
---

Choosing the correct hyperparameters for machine learning or deep learning models is one of the best ways to extract the last juice out of your models.


## [The difference between parameter and hyperparameter]({{< relref "2024-05-20-012525-the_difference_between_parameter_and_hyperparameter.md" >}}) {#the-difference-between-parameter-and-hyperparameter--2024-05-20-012525-the-difference-between-parameter-and-hyperparameter-dot-md}


## What is hyperparameter tuning and why it is important? {#what-is-hyperparameter-tuning-and-why-it-is-important}

Hyperparameter tuning (or hyperparameter optimization) is the process of determining the right combination of hyperparameters that maximizes the model performance. It works by running multiple trials in a single training process. Each trial is a complete execution of your training application with values for your chosen hyperparameters, set within the limits you specify. This process once finished will give you the set of hyperparameter values that are best suited for the model to give optimal results.

Needless to say, It is an important step in any Machine Learning project since it leads to optimal results for a model. If you wish to see it in action, here’s a research paper that talks about the importance of hyperparameter optimization by experimenting on datasets.
<https://arxiv.org/pdf/2007.07588>
<https://towardsdatascience.com/hyperparameter-tuning-c5619e7e6624>


## How to do hyperparameter tuning? How to find the best hyperparameters? {#how-to-do-hyperparameter-tuning-how-to-find-the-best-hyperparameters}

Choosing the right combination of hyperparameters requires an understanding of the hyperparameters and the business use-case. However, technically, there are two ways to set them.


### Manual hyperparameter tuning {#manual-hyperparameter-tuning}

Manual hyperparameter tuning involves experimenting with different sets of hyperparameters manually i.e. each trial with a set of hyperparameters will be performed by you. This technique will require a robust experiment tracker which could track a variety of variables from images, logs to system metrics.

There are a few experiment trackers that tick all the boxes. [neptune.ai](https://neptune.ai/) is one of them. It offers an intuitive interface and an open-source package neptune-client to facilitate logging into your code. You can easily log hyperparameters and see all types of data results like images, metrics, etc. Head over to the docs to see how you can [log different metadata to Neptune](https://docs.neptune.ai/logging/what_you_can_log/).

Alternative solutions include W&amp;B, Comet, or [MLFlow]({{< relref "2023-11-30-224657-mlflow.md" >}}). [Check more tools for experiment tracking &amp; management here](https://neptune.ai/blog/best-ml-experiment-tracking-tools).


#### Advantages of manual hyperparameter optimization: {#advantages-of-manual-hyperparameter-optimization}

-   Tuning hyperparameters manually means more control over the process.
-   If you’re researching or studying tuning and how it affects the network weights then doing it manually would make sense.


#### Disadvantages of manual hyperparameter optimization: {#disadvantages-of-manual-hyperparameter-optimization}

-   Manual tuning is a tedious process since there can be many trials and keeping track can prove costly and time-consuming.
-   This isn’t a very practical approach when there are a lot of hyperparameters to consider.

Read about [how to manually optimize Machine Learning model hyperparameters here](https://machinelearningmastery.com/manually-optimize-hyperparameters/).


### Automated hyperparameter tuning {#automated-hyperparameter-tuning}

Automated hyperparameter tuning utilizes already existing algorithms to automate the process. The steps you follow are:

-   First, specify a set of hyperparameters and limits to those hyperparameters’ values (note: every algorithm requires this set to be a specific data structure, e.g. dictionaries are common while working with algorithms).
-   Then the algorithm does the heavy lifting for you. It runs those trials and fetches you the best set of hyperparameters that will give optimal results.

In the blog, we will talk about some of the algorithms and tools you could use to achieve automated tuning. Let’s get to it.


## Hyperparameter tuning methods {#hyperparameter-tuning-methods}

In this section, I will introduce all of the hyperparameter optimization methods that are popular today.


### Random Search {#random-search}

In the [random search method](https://www.jmlr.org/papers/volume13/bergstra12a/bergstra12a.pdf), we create a grid of possible values for hyperparameters. Each iteration tries a random combination of hyperparameters from this grid, records the performance, and lastly returns the combination of hyperparameters that provided the best performance.


### Grid Search {#grid-search}

In the [grid search method](https://towardsdatascience.com/grid-search-for-model-tuning-3319b259367e), we create a grid of possible values for hyperparameters. Each iteration tries a combination of hyperparameters in a specific order. It fits the model on each and every combination of hyperparameters possible and records the model performance. Finally, it returns the best model with the best hyperparameters.
![](https://i0.wp.com/neptune.ai/wp-content/uploads/2022/10/grid_random.png?resize=569%2C301&ssl=1)


### Bayesian Optimization {#bayesian-optimization}

Tuning and finding the right hyperparameters for your model is an optimization problem. We want to minimize the loss function of our model by changing model parameters. Bayesian optimization helps us find the minimal point in the minimum number of steps. [Bayesian optimization](https://towardsdatascience.com/a-conceptual-explanation-of-bayesian-model-based-hyperparameter-optimization-for-machine-learning-b8172278050f) also uses an acquisition function that directs sampling to areas where an improvement over the current best observation is likely.


### Tree-structured Parzen estimators (TPE) {#tree-structured-parzen-estimators--tpe}

The idea of [Tree-based Parzen optimization](https://optunity.readthedocs.io/en/latest/user/solvers/TPE.html) is similar to Bayesian optimization. Instead of finding the values of p(y|x) where y is the function to be minimized (e.g., validation loss) and x is the value of hyperparameter the TPE models P(x|y) and P(y). One of the great drawbacks of tree-structured Parzen estimators is that they do not model interactions between the hyper-parameters. That said TPE works extremely well in practice and was battle-tested across most domains.


## Hyperparameter tuning algorithms {#hyperparameter-tuning-algorithms}

These are the algorithms developed specifically for doing hyperparameter tuning.


### Hyperband {#hyperband}

Hyperband is a variation of random search, but with some explore-exploit theory to find the best time allocation for each of the configurations. You can check this [research paper](https://arxiv.org/abs/1603.06560) for further references.


### Population-based training (PBT) {#population-based-training--pbt}

This technique is a hybrid of the two most commonly used search techniques: Random Search and manual tuning applied to Neural Network models.

PBT starts by training many neural networks in parallel with random hyperparameters. But these networks aren’t fully independent of each other.

It uses information from the rest of the population to refine the hyperparameters and determine the value of hyperparameter to try. You can check this [article](https://deepmind.google/discover/blog/population-based-training-of-neural-networks/) for more information on PBT.
![](https://i0.wp.com/neptune.ai/wp-content/uploads/2022/10/population-based-training.png?ssl=1)


### BOHB {#bohb}

BOHB (Bayesian Optimization and HyperBand) mixes the Hyperband algorithm and Bayesian optimization. You can check this [article](https://www.automl.org/blog_bohb/) for further reference.
<https://neptune.ai/blog/hyperband-and-bohb-understanding-state-of-the-art-hyperparameter-optimization-algorithms>


## Tools for hyperparameter optimization {#tools-for-hyperparameter-optimization}

Now that you know what are the methods and algorithms let’s talk about tools, and there are a lot of those out there.

Some of the best hyperparameter optimization libraries are:

-   [optuna]({{< relref "2024-05-18-232947-optuna.md" >}})


## Reference List {#reference-list}

1.  <https://neptune.ai/blog/hyperparameter-tuning-in-python-complete-guide>
