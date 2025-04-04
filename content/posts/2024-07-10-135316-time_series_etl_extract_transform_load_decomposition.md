---
title: "Time series ETL (Extract, Transform, Load) Decomposition"
draft: false
---

[Time Series]({{< relref "20230404184340-time_series.md" >}}) ETL (Extract, Transform, Load) Decomposition is a process used to analyze and prepare time series data for further analysis, such as forecasting or anomaly detection. It involves breaking down a time series into its constituent components to better understand the underlying patterns and trends. Here’s a breakdown of the main components of time series decomposition:

-   Extraction (E):

Extracting Data: This involves collecting time series data from various sources. The data might come from databases, APIs, sensors, logs, etc. This step ensures that the data is gathered in a raw form, maintaining its temporal order.
Transformation (T):

-   Decomposition: This is the core step in time series decomposition. The time series data is divided into several components, typically:

Trend Component: Represents the long-term progression or direction in the data.
Seasonal Component: Captures the repeating patterns or cycles within specific periods (e.g., daily, monthly, yearly).
Residual (or Irregular) Component: The remaining part of the data after removing the trend and seasonal components, representing random noise or anomalies.
Smoothing: Techniques like moving averages or exponential smoothing might be applied to highlight the trend and seasonal components more clearly.
Normalization/Standardization: Adjusting the scale of the data to make it more suitable for analysis.
Imputation: Handling missing values in the time series data.

-   Loading (L):

Loading Data: After transforming and decomposing the data, it is loaded into a target system, such as a data warehouse, database, or a data visualization tool for further analysis and modeling.


## Methods of Time Series Decomposition {#methods-of-time-series-decomposition}

1.  Classical Decomposition:
    -   Additive Model: Assumes that the components add together linearly: \\(Y(t)=T(t)+S(t)+R(t)\\)
    -   Multiplicative Model: Assumes that the components multiply together: \\[Y(t)=T(t)×S(t)×R(t)\\]
2.  STL (Seasonal and Trend Decomposition using Loess):
3.  A robust and flexible method that decomposes time series data using locally weighted regression (Loess).
4.  X-12-ARIMA/X-13-ARIMA-SEATS:
5.  Methods developed by statistical agencies for seasonal adjustment and decomposition of time series data.

<!--listend-->

```python
 from statsmodels.tsa.seasonal import seasonal_decompose
import matplotlib.pyplot as plt
 # Perform decomposition
 decomposition = seasonal_decompose(df['Energy'], model='additive', period=12)
 # Create custom subplots for each component
 # Initialize a figure with 4 subplots arranged vertically
 fig, axes = plt.subplots(4, 1, figsize=(12, 8), sharex=True)
 # Plot the observed component in the first subplot
 decomposition.observed.plot(ax=axes[0], color='green')
 axes[0].set_ylabel('Observed')
 # Plot the trend component in the second subplot
 decomposition.trend.plot(ax=axes[1], color='green')
 axes[1].set_ylabel('Trend')
 # Plot the seasonal component in the third
 decomposition.seasonal.plot(ax=axes[2], color='green')
 axes[2].set_ylabel('Seasonal')
 # Plot the residual component in the fourth subplot
 decomposition.resid.plot(ax=axes[3], color='green')
 axes[3].set_ylabel('Residual')
 # Set a common X-axis label
 plt.xlabel('Date')
 # Set the main title for the entire plot
 plt.suptitle('ETS Decomposition', fontsize=16)
 # Adjust subplot spacing and position the main title
 plt.tight_layout()
 plt.subplots_adjust(top=0.9)
 # Display the plot
 plt.show()
```

**NOTE**:


### Model {#model}

This parameter specifies the type of decomposition model to be used.


#### Additive Model {#additive-model}

The 'additive' model assumes that the time series can be expressed as the sum of its components:
\\[Y(t)=T(t)+S(t)+R(t)\\]
where \\(Y(t)\\) is the observed value, \\(T(t)\\) is the trend component, \\(S(t)\\) is the seasonal component, and \\(R(t)\\) is the residual component.


#### Multiplicative Model {#multiplicative-model}

The multiplicative model assumes that the components multiply together to make the time series. It is appropriate when the variation around the trend changes with the level of the time series.
\\[Y(t)=T(t)×S(t)×R(t)\\]
In this model, the components interact in a multiplicative manner, making it suitable for time series where the seasonal variation increases or decreases with the level of the trend.


### Period {#period}

This parameter specifies the periodicity of the seasonal component. In this case, a period of 12 indicates that the seasonal pattern repeats every 12 time units. This is common for monthly data with an annual seasonality (e.g., if you have monthly data, period=12 would correspond to yearly seasonality).


## Reference List {#reference-list}

1.  <https://medium.com/analytics-vidhya/assessment-of-accuracy-metrics-for-time-series-forecasting-bc115b655705>
