---
title: "Smoothing Techniques for time series data"
draft: false
---

[Time Series]({{< relref "20230404184340-time_series.md" >}})

[Data Smoothing]({{< relref "20230516154402-data_smoothing.md" >}})


## Introduction {#introduction}

Smoothing techniques are kinds of data preprocessing techniques to remove noise from a data set. This allows important patterns to stand out.


## Moving average (MA) smoothing {#moving-average--ma--smoothing}

It is a simple and common type of smoothing used in time series analysis and forecasting. Here time series derived from the average of last kth elements of the series.

The formula provided is:
\\[\\[ S\_t = \frac{X\_{t-k} + X\_{t-k+1} + X\_{t-k+2} + \ldots + X\_t}{k} \\]\\]
Where:

-   \\(S\_t\\) is the smoothed value at time \\(t\\).
-   \\(X\_t\\) is the actual value at time \\(t\\).
-   \\(k\\) is the number of periods over which the average is calculated.

<!--listend-->

```python
def moving_avarage_smoothing(X,k):
        S = np.zeros(X.shape[0])
        for t in range(X.shape[0]):
                if t < k:
                        S[t] = np.mean(X[:t+1])
                else:
                        S[t] = np.sum(X[t-k:t])/k
        return S
```


## Exponential smoothing {#exponential-smoothing}

Exponential smoothing is a weighted moving average technique. In the moving average smoothing the past observations are weighted equally, In this case smoothing is done by assigning exponentially decreasing weights to the past observations.

\\[
S\_0 = X\_0
\\]

\\[
S\_t = \alpha \cdot X\_t + (1 - \alpha) \cdot S\_{t-1} \quad \\{ t > 0, \\, 0 < \alpha < 1 \\}
\\]

In the above equation, we can see that \\((1 - \alpha)\\) is multiplied by the previously expected value \\(S\_{t-1}\\) which is derived using the same formula. This makes the expression recursive, and if you were to write it all out on paper, you would quickly see that \\((1 - \alpha)\\) is multiplied by itself again and again. And this is why this method is called exponential.

```python
def exponential_smoothing(X,α):
        S = np.zeros(X.shape[0])
        S[0] = X[0]
        for t in range(1,X.shape[0]):
                S[t] = α * X[t-1] + (1- α) * S[t-1]
        return S
```


## Double exponential smoothing {#double-exponential-smoothing}

Single Smoothing does not excel in the data when there is a trend. This situation can be improved by the introduction of a second equation with a second constant β.

t is suitable to model the time series with the trend but without seasonality.

\\[
S\_0 = X\_0
\\]

\\[
B\_0 = X\_1 - X\_0
\\]

\\[
S\_t = \alpha \cdot X\_t + (1 - \alpha) \cdot (S\_{t-1} + B\_{t-1})
\\]

\\[
B\_t = \beta \cdot (S\_t - S\_{t-1}) + (1 - \beta) \cdot B\_{t-1}
\\]

\\[
\alpha, \beta \in (0, 1)
\\]

Here it is seen that \\(\alpha\\) is used for smoothing the level and \\(\beta\\) is used for smoothing the trend.

```python
def double_exponential_smoothing(X,α,β):
        S,A,B = (np.zeros( X.shape[0] ) for i in range(3))
        S[0] = X[0]
        B[0] = X[1] - X[0]
        for t in range(1,X.shape[0]):
                A[t] = α * X[t] + (1- α) * S[t-1]
                B[t] = β * (A[t] - A[t-1]) + (1 - β) * B[t-1]
                S[t] = A[t] + B[t]
        return S
```


## Triple exponential smoothing {#triple-exponential-smoothing}

It is also called as Holt-winters exponential smoothing .it is used to handle the time series data containing a seasonal component.

double smoothing will not work in case of data contain seasonality.so that for smoothing the seasonality a third equation is introduced.

\\[
S\_0, F\_0 = X\_0
\\]

\\[
B\_0 = \frac{\sum\_{i=0}^{L-1} (X\_{L+i} - X\_i)}{L^2}
\\]

\\[
S\_t = \alpha \cdot (X\_t - C\_{\\%L}) + (1 - \alpha) \cdot (S\_{t-1} + \phi \cdot B\_{t-1})
\\]

\\[
B\_t = \beta \cdot (S\_t - S\_{t-1}) + (1 - \beta) \cdot \phi \cdot B\_{t-1}
\\]

\\[
C\_{\\%L} = \gamma \cdot (X\_t - S\_t) + (1 - \gamma) \cdot C\_{\\%L}
\\]

\\[
F\_{t+m} = S\_t + B\_t \cdot \sum\_{i=1}^{m} \phi^i + C\_{\\%L}
\\]

\\[
\alpha, \beta, \gamma \in (0, 1)
\\]

In the above, \\(\phi\\) is the damping constant. \\(\alpha\\), \\(\beta\\), and \\(\gamma\\) must be estimated in such a way that the [Root Mean Squared Error (RMSE)]({{< relref "2024-07-10-131415-assessment_of_accuracy_metrics_for_time_series_forecasting.md#root-mean-squared-error--rmse" >}}) of the error is minimized.

```python
def triple_exponential_smoothing(X, L, α, β, γ, ϕ, m):
    def sig_ϕ(ϕ, m):
        return np.sum(np.array([np.power(ϕ, i) for i in range(m + 1)]))

    C, S, B, F = (np.zeros(X.shape[0]) for _ in range(4))
    S[0], F[0] = X[0], X[0]
    B[0] = np.mean(X[L:2*L] - X[:L]) / L
    sig_ϕ_val = sig_ϕ(ϕ, m)

    for t in range(1, X.shape[0]):
        S[t] = α * (X[t] - C[t % L]) + (1 - α) * (S[t-1] + ϕ * B[t-1])
        B[t] = β * (S[t] - S[t-1]) + (1-β) * ϕ * B[t-1]
        C[t % L] = γ * (X[t] - S[t]) + (1 - γ) * C[t % L]
        F[t] = S[t] + sig_ϕ_val * B[t] + C[t % L]

    return S
```

**NOTE**:

-   X is data
-   L is Season Length The season length L represents the number of periods in a full seasonal cycle. For example, if the data is monthly and exhibits yearly seasonality, L would be 12.
-   S represents the smoothed value (level).
-   B represents the trend component.
-   C represents the seasonal component.
-   F represents the forecast value.
-   α, β, and γ are the smoothing constants for the level, trend, and seasonality, respectively.
-   ϕ is the damping factor.
-   The variable m in the context of the triple exponential smoothing function typically represents the number of future periods for which the forecast is being made.


## Reference List {#reference-list}

1.  <https://medium.com/@srv96/smoothing-techniques-for-time-series-data-91cccfd008a2>
