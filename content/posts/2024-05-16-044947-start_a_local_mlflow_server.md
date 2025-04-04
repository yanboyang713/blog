---
title: "start a local MLflow server"
draft: false
---

<https://towardsdatascience.com/deploy-mlflow-with-docker-compose-8059f16b6039>


## Start a Local MLflow Server {#start-a-local-mlflow-server}

[mamba]({{< relref "20230622162323-mamba.md" >}})


### Create environment and install mlflow {#create-environment-and-install-mlflow}

```bash
mamba create -n mlflow jupyterlab -c conda-forge
mamba activate mlflow
pip install mlflow
```


### start a local MLflow server with UI by running the command below {#start-a-local-mlflow-server-with-ui-by-running-the-command-below}

```bash
mlflow ui
```

Or,

```bash
mlflow server --host 127.0.0.1 --port 8080
```

There are many options to configure the server, refer to <https://mlflow.org/docs/latest/tracking/server.html#configure-server>


### Run jupyter lab {#run-jupyter-lab}

```bash
jupyter lab
```


### Create a new notebook {#create-a-new-notebook}


### Connect MLflow Session to Your Server {#connect-mlflow-session-to-your-server}

Now that the server is spun up, let’s connect our MLflow session to the local server.

```python
import mlflow
mlflow.set_tracking_uri("http://127.0.0.1:5000")
```

Next, let’s try logging some dummy metrics. We can view these test metrics on the local hosted UI.
Test connection

```python
mlflow.set_experiment("/check-localhost-connection")

with mlflow.start_run():
    mlflow.log_metric("foo", 1)
    mlflow.log_metric("bar", 2)
```

Another, example for [python]({{< relref "20230404161934-python.md" >}})

```python
import mlflow

remote_server_uri = "..."  # set to your server URI
mlflow.set_tracking_uri(remote_server_uri)
mlflow.set_experiment("/my-experiment")
with mlflow.start_run():
    mlflow.log_param("a", 1)
    mlflow.log_metric("b", 2)
```

Another, example for [r language]({{< relref "20230622162146-r_language.md" >}})

```r
library(mlflow)
install_mlflow()
remote_server_uri = "..." # set to your server URI
mlflow_set_tracking_uri(remote_server_uri)
mlflow_set_experiment("/my-experiment")
mlflow_log_param("a", "1")
```


### View Experiment on Your MLflow Server {#view-experiment-on-your-mlflow-server}

Now let’s view your experiment on the local server. Open the URL in your browser, which is <http://localhost:5000> in our case. In the UI, inside the left sidebar you should see the experiment with name “check-localhost-connection”. Clicking on this experiment name should bring you to the experiment view


## Reference List {#reference-list}

1.  <https://mlflow.org/docs/latest/getting-started/tracking-server-overview/index.html#start-a-local-mlflow-server>
