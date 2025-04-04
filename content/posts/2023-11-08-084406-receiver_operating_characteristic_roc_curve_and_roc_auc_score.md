---
title: "receiver operating characteristic (ROC) curve and roc auc score"
draft: false
---

ROC curves are VERY help with understanding the balance between true-positive rate and false positive rates. Sci-kit learn has built in functions for ROC curves and for analyzing them. The inputs to these functions (roc_curve and roc_auc_score) are the actual labels and the predicted probabilities (not the predicted labels). Both roc_curve and roc_auc_score are both complicated functions, so we will not have you write these functions from scratch. Instead, we will show you how to use sci-kit learn's functions and explain the key points. Let's begin by using roc_curve to make the ROC plot.

```python
from sklearn.metrics import roc_curve
fpr_RF, tpr_RF, thresholds_RF = roc_curve(df.actual_label.values, df.model_RF.values)
fpr_LR, tpr_LR, thresholds_LR = roc_curve(df.actual_label.values, df.model_LR.values)
```

The roc_curve function returns three lists:

-   thresholds = all unique prediction probabilities in descending order
-   fpr = the false positive rate (FP / (FP + TN)) for each threshold
-   tpr = the true positive rate (TP / (TP + FN)) for each threshold

We can plot the ROC curve for each model as shown below.

```python
import matplotlib.pyplot as plt
plt.plot(fpr_RF, tpr_RF,'r-',label = 'RF')
plt.plot(fpr_LR,tpr_LR,'b-', label= 'LR')
plt.plot([0,1],[0,1],'k-',label='random')
plt.plot([0,0,1,1],[0,1,1,1],'g-',label='perfect')
plt.legend()
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.show()
```

There are a couple things that we can observer from this figure:

-   a model that randomly guesses the label will result in the black line and you want to have a model that has a curve above this black line
-   an ROC that is farther away from the black line is better, so RF (red) looks better than LR (blue)
-   Although not seen directly, a high threshold results in a point in the bottom left and a low threshold results in a point in the top right. This means as you decrease the threshold you get higher TPR at the cost of a higher FPR

To analyze the performance, we will use the area-under-curve metric.

```python
from sklearn.metrics import roc_auc_score
auc_RF = roc_auc_score(df.actual_label.values, df.model_RF.values)
auc_LR = roc_auc_score(df.actual_label.values, df.model_LR.values)
print('AUC RF:%.3f'% auc_RF)
print('AUC LR:%.3f'% auc_LR)
```

As you can see, the area under the curve for the RF model (AUC = 0.738) is better than the LR (AUC = 0.666). When I plot the ROC curve, I like to add the AUC to the legend as shown below.

```python
import matplotlib.pyplot as plt
plt.plot(fpr_RF, tpr_RF,'r-',label = 'RF AUC: %.3f'%auc_RF)
plt.plot(fpr_LR,tpr_LR,'b-', label= 'LR AUC: %.3f'%auc_LR)
plt.plot([0,1],[0,1],'k-',label='random')
plt.plot([0,0,1,1],[0,1,1,1],'g-',label='perfect')
plt.legend()
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.show()
```

Overall, in this toy example the model RF wins with every performance metric.


## Reference List {#reference-list}

1.  <https://towardsdatascience.com/understanding-data-science-classification-metrics-in-scikit-learn-in-python-3bc336865019>
