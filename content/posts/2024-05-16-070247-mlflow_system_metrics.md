---
title: "MLflow System Metrics"
draft: false
---

[MLFlow]({{< relref "2023-11-30-224657-mlflow.md" >}}) allows users to log system metrics including CPU stats, [GPU]({{< relref "2023-10-30-193539-gpu.md" >}}) stats, memory usage, [networking measurement metrics]({{< relref "20230609162126-networking_measurement_metrics.md" >}}), and disk usage during the execution of an MLflow run.


## Extra Dependencies {#extra-dependencies}

To log system metrics in MLflow, please install psutil. To install psutil, run the following command:

```bash
pip install psutil
```

If you want to catch GPU metrics, you also need to install pynvml:

```bash
pip install pynvml
```


## Turn on/off System Metrics Logging {#turn-on-off-system-metrics-logging}

There are three ways to enable or disable system metrics logging:


### Using the Environment Variable to Control System Metrics Logging {#using-the-environment-variable-to-control-system-metrics-logging}

You can set the environment variable **MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING** to true to turn on system metrics logging globally, as shown below:

```bash
export MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING=true
```

However, if you are executing the command above from within Ipython notebook (Jupyter, Databricks notebook, Google Colab), the export command will not work due to the segregated state of the ephemeral shell. Instead you can use the following code:

```bash
import os

os.environ["MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING"] = "true"
```

After setting the environment variable, you will see that starting an MLflow run will automatically collect and log the default system metrics. Try running the following code in your favorite environment and you should see system metrics existing in the logged run data. Please note that you don’t necessarilty need to start an MLflow server, as the metrics are logged locally.

```python
import mlflow
import time

with mlflow.start_run() as run:
    time.sleep(15)

print(mlflow.MlflowClient().get_run(run.info.run_id).data)
```

Your output should look like this:

```file
<RunData: metrics={'system/cpu_utilization_percentage': 12.4,
'system/disk_available_megabytes': 213744.0,
'system/disk_usage_megabytes': 28725.3,
'system/disk_usage_percentage': 11.8,
'system/network_receive_megabytes': 0.0,
'system/network_transmit_megabytes': 0.0,
'system/system_memory_usage_megabytes': 771.1,
'system/system_memory_usage_percentage': 5.7}, params={}, tags={'mlflow.runName': 'nimble-auk-61',
'mlflow.source.name': '/usr/local/lib/python3.10/dist-packages/colab_kernel_launcher.py',
'mlflow.source.type': 'LOCAL',
'mlflow.user': 'root'}>
```

To disable system metrics logging, you can use either of the following commands:

```bash
export MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING="false"
```

```python
import os

del os.environ["MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING"]
```

Rerunning the MLflow code above will not log system metrics.


### Using **mlflow.enable_system_metrics_logging()** {#using-mlflow-dot-enable-system-metrics-logging}

We also provide a pair of APIs **mlflow.enable_system_metrics_logging()** and **mlflow.disable_system_metrics_logging()** to turn on/off system metrics logging globally for environments in which you do not have the appropriate access to set an environment variable. Running the following code will have the same effect as setting **MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING** environment variable to true:

```bash
import mlflow

mlflow.enable_system_metrics_logging()

with mlflow.start_run() as run:
    time.sleep(15)

print(mlflow.MlflowClient().get_run(run.info.run_id).data)
```


### Enabling System Metrics Logging for a Single Run {#enabling-system-metrics-logging-for-a-single-run}

In addition to controlling system metrics logging globally, you can also control it for a single run. To do so, set **log_system_metrics** as True or False accordingly in **mlflow.start_run()**:

```python
with mlflow.start_run(log_system_metrics=True) as run:
    time.sleep(15)

print(mlflow.MlflowClient().get_run(run.info.run_id).data)
```

Please also note that using **log_system_metrics** will ignore the global status of system metrics logging. In other words, the above code will log system metrics for the specific run even if you have disabled system metrics logging by setting **MLFLOW_ENABLE_SYSTEM_METRICS_LOGGING** to false or calling **mlflow.disable_system_metrics_logging()**.


## Reference List {#reference-list}

1.  <https://mlflow.org/docs/latest/system-metrics/index.html#system-metrics>
