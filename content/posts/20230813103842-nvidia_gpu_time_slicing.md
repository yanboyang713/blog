---
title: "nvidia GPU Time-slicing"
draft: false
---

## Time-slicing {#time-slicing}

When dealing with multiple CUDA applications, each of which may not fully utilize the GPU’s resources, you can use a simple oversubscription strategy to leverage the GPU’s time-slicing scheduler. This is supported by compute preemption starting with the Pascal architecture. This technique, sometimes called temporal GPU sharing, does carry a cost for context-switching between the different CUDA applications, but some underutilized applications can still benefit from this strategy.

Since CUDA 11.1 (R455+ drivers), the time-slice duration for CUDA applications is configurable through the nvidia-smi utility:

```console
$ nvidia-smi compute-policy --help

    Compute Policy -- Control and list compute policies.

    Usage: nvidia-smi compute-policy [options]

    Options include:
    [-i | --id]: GPU device ID's. Provide comma
                 separated values for more than one device

    [-l | --list]: List all compute policies

    [ | --set-timeslice]: Set timeslice config for a GPU:
                          0=DEFAULT, 1=SHORT, 2=MEDIUM, 3=LONG

    [-h | --help]: Display help information
```

The tradeoffs with time-slicing are increased latency, jitter, and potential out-of-memory (OOM) conditions when many different applications are time-slicing on the GPU. This mechanism is what we focus on in the second part of this post.


### Time-slicing support in [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) {#time-slicing-support-in-kubernetes--20230105185343-kubernetes-dot-md}

NVIDIA GPUs are advertised as schedulable resources in Kubernetes through the device plugin framework. However, this framework only allows for devices, including GPUs (as nvidia.com/gpu)  to be advertised as integer resources and thus does not allow for oversubscription. In this section, we discuss a new method for oversubscribing GPUs in Kubernetes using time-slicing.

Before we discuss the new APIs, we introduce a new mechanism for configuring the NVIDIA Kubernetes device plugin using a configuration file.


#### New configuration file support {#new-configuration-file-support}

The Kubernetes device plugin offers a number of options for configuration, which are set either as command-line options or environment variables, such as setting MIG strategy, device enumeration, and so on. Similarly, gpu-feature-discovery (GFD) uses a similar option for generating labels to describe GPU nodes.

As configuration options become more complex, you use a configuration file to express these options to the Kubernetes device plugin and GFD, which is then deployed as a configmap object and applied to the plugin and the GFD pods during startup.

The configuration options are expressed in a YAML file. In the following example, you record the various options in a file called dp-example-config.yaml, created under /tmp.

```yaml
$ cat << EOF > /tmp/dp-example-config.yaml
version: v1
flags:
  migStrategy: "none"
  failOnInitError: true
  nvidiaDriverRoot: "/"
  plugin:
    passDeviceSpecs: false
    deviceListStrategy: "envvar"
    deviceIDStrategy: "uuid"
  gfd:
    oneshot: false
    noTimestamp: false
    outputFile: /etc/kubernetes/node-feature-discovery/features.d/gfd
    sleepInterval: 60s
EOF
```

Then, start the Kubernetes device plugin by specifying the location of the config file and using the gfd.enabled=true option to start GFD as well:

```bash
$ helm install nvdp nvdp/nvidia-device-plugin \
    --version=0.12.2 \
    --namespace nvidia-device-plugin \
    --create-namespace \
    --set gfd.enabled=true \
    --set-file config.map.config=/tmp/dp-example-config.yaml
```


#### Dynamic configuration changes {#dynamic-configuration-changes}

The configuration is applied to all GPUs on all nodes by default. The Kubernetes device plugin enables multiple configuration files to be specified. You can override the configuration on a node-by-node basis by overwriting a label on the node.

The Kubernetes device plugin uses a sidecar container that detects changes in desired node configurations and reloads the device plugin so that new configurations can take effect. In the following example, you create two configurations for the device plugin: a default that is applied to all nodes and another that you can apply to A100 GPU nodes on demand.

```bash
$ helm install nvdp nvdp/nvidia-device-plugin \
    --version=0.12.2 \
    --namespace nvidia-device-plugin \
    --create-namespace \
    --set gfd.enabled=true \
    --set-file config.map.default=/tmp/dp-example-config-default.yaml \
    --set-file config.map.a100-80gb-config=/tmp/dp-example-config-a100.yaml
```

The Kubernetes device plugin then enables dynamic changes to the configuration whenever the node label is overwritten, allowing for configuration on a per-node basis if so desired:

```bash
$ kubectl label node \
   --overwrite \
   --selector=nvidia.com/gpu.product=A100-SXM4-80GB \
    nvidia.com/device-plugin.config=a100-80gb-config
```


#### Time-slicing APIs {#time-slicing-apis}

To support time-slicing of GPUs, you extend the definition of the configuration file with the following fields:

```yaml
version: v1
sharing:
  timeSlicing:
    renameByDefault: <bool>
    failRequestsGreaterThanOne: <bool>
    resources:
    - name: <resource-name>
      replicas: <num-replicas>
    ...
```

That is, for each named resource under sharing.timeSlicing.resources, a number of replicas can now be specified for that resource type.

Moreover, if renameByDefault=true, then each resource is advertised under the name &lt;resource-name&gt;.shared instead of simply &lt;resource-name&gt;.

The failRequestsGreaterThanOne flag is false by default for backward compatibility. It controls whether pods can request more than one GPU resource. A request of more than one GPU does not imply that the pod gets proportionally more time slices, as the GPU scheduler currently gives an equal share of time to all processes running on the GPU.

The failRequestsGreaterThanOne flag configures the behavior of the plugin to treat a request of one GPU as an access request rather than an exclusive resource request.

As the new oversubscribed resources are created, the Kubernetes device plugin assigns these resources to the requesting jobs. When two or more jobs land on the same GPU, the jobs automatically use the GPU’s time-slicing mechanism. The plugin does not offer any other additional isolation benefits.


#### Labels applied by GFD {#labels-applied-by-gfd}

For GFD, the labels that get applied depend on whether renameByDefault=true. Regardless of the setting for renameByDefault,  the following label is always applied:

```bash
nvidia.com/<resource-name>.replicas = <num-replicas>
```

However, when renameByDefault=false, the following suffix is also added to the nvidia.com/&lt;resource-name&gt;.product label:

```bash
nvidia.com/gpu.product = <product name>-SHARED
```

Using these labels, you have a way of selecting a shared or non-shared GPU in the same way you would traditionally select one GPU model over another. That is, the SHARED annotation ensures that you can use a nodeSelector object to attract pods to nodes that have shared GPUs on them. Moreover, the pods can ensure that they land on a node that is dividing a GPU into their desired proportions using the new replicas label.


#### Oversubscribing example {#oversubscribing-example}

Here’s a complete example of oversubscribing GPU resources using the time-slicing APIs. In this example, you walk through the additional configuration settings for the Kubernetes device plugin and GFD) to set up GPU oversubscription and launch a workload using the specified resources.

Consider the following configuration file:

```yaml
version: v1
sharing:
  timeSlicing:
    resources:
    - name: nvidia.com/gpu
      replicas: 5
    ...
```

If this configuration were applied to a node with eight GPUs on it, the plugin would now advertise 40 nvidia.com/gpu resources to Kubernetes instead of eight. If the renameByDefault: true option was set, then 40 nvidia.com/gpu.shared resources would be advertised instead of eight nvidia.com/gpu resources.

You enable time-slicing in the following example configuration. In this example, oversubscribe the GPUs by 2x:

```console
$ cat << EOF > /tmp/dp-example-config.yaml
version: v1
flags:
  migStrategy: "none"
  failOnInitError: true
  nvidiaDriverRoot: "/"
  plugin:
    passDeviceSpecs: false
    deviceListStrategy: "envvar"
    deviceIDStrategy: "uuid"
  gfd:
    oneshot: false
    noTimestamp: false
    outputFile: /etc/kubernetes/node-feature-discovery/features.d/gfd
    sleepInterval: 60s
sharing:
  timeSlicing:
    resources:
    - name: nvidia.com/gpu
      replicas: 2
EOF
```

Set up the Helm chart repository:

```bash
$ helm repo add nvdp https://nvidia.github.io/k8s-device-plugin \
   && helm repo update
```

Now, deploy the Kubernetes device plugin by specifying the location to the config file created earlier:

```bash
$ helm install nvdp nvdp/nvidia-device-plugin \
    --version=0.12.2 \
    --namespace nvidia-device-plugin \
    --create-namespace \
    --set gfd.enabled=true \
    --set-file config.map.config=/tmp/dp-example-config.yaml
```

As the node only has a single physical GPU, you can now see that the device plugin advertises two GPUs as allocatable:

```console
$ kubectl describe node
...
Capacity:
  cpu:                4
  ephemeral-storage:  32461564Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16084408Ki
  nvidia.com/gpu:     2
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  29916577333
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             15982008Ki
  nvidia.com/gpu:     2
  pods:               110
```

Next, deploy two applications (in this case, an FP16 CUDA GEMM workload) with each requesting one GPU. Observe that the applications context switch on the GPU and thus only achieve approximately half the peak FP16 bandwidth on a T4.

```console
$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: dcgmproftester-1
spec:
  restartPolicy: "Never"
  containers:
  - name: dcgmproftester11
    image: nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04
    args: ["--no-dcgm-validation", "-t 1004", "-d 30"]
    resources:
      limits:
         nvidia.com/gpu: 1
    securityContext:
      capabilities:
        add: ["SYS_ADMIN"]

---

apiVersion: v1
kind: Pod
metadata:
  name: dcgmproftester-2
spec:
  restartPolicy: "Never"
  containers:
  - name: dcgmproftester11
    image: nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04
    args: ["--no-dcgm-validation", "-t 1004", "-d 30"]
    resources:
      limits:
         nvidia.com/gpu: 1
    securityContext:
      capabilities:
        add: ["SYS_ADMIN"]
EOF
```

You can now see the two containers deployed and running on a single physical GPU, which would not have been possible in Kubernetes without the new time-slicing APIs:

```console
$ kubectl get pods -A
NAMESPACE              NAME                                                              READY   STATUS    RESTARTS   AGE
default                dcgmproftester-1                                                  1/1     Running   0          45s
default                dcgmproftester-2                                                  1/1     Running   0          45s
kube-system            calico-kube-controllers-6fcb5c5bcf-cl5h5                          1/1     Running   3          32d
```

You can use nvidia-smi on the host to see that the two containers are scheduled on the same physical GPU by the plugin and context switch on the GPU:

```console
$ nvidia-smi -L
GPU 0: Tesla T4 (UUID: GPU-491287c9-bc95-b926-a488-9503064e72a1)

$ nvidia-smi
...<snip>...

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A    466420      C   /usr/bin/dcgmproftester11         315MiB |
|    0   N/A  N/A    466421      C   /usr/bin/dcgmproftester11         315MiB |
+-----------------------------------------------------------------------------+
```


## Reference List {#reference-list}

1.  <https://developer.nvidia.com/blog/improving-gpu-utilization-in-kubernetes/>
