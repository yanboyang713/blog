---
title: "NodePort"
tags: ["definition"]
draft: false
---

## Definition {#definition}

A NodePort publicly exposes a service on a fixed port number. It lets you access the service from outside your cluster. You’ll need to use the cluster’s IP address and the NodePort number—e.g. 123.123.123.123:30000.

Creating a NodePort will open that port on every node in your cluster. Kubernetes will automatically route port traffic to the service it’s linked to.


## Examples {#examples}

Here’s an example NodePort service manifest:

```yaml
apiVersion: v1
kind: Service
spec:
  selector:
    app: my-app
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
```

NodePort definitions have the same mandatory properties as [ClusterIP]({{< relref "20221124055917-clusterip.md" >}}) services. The only difference is the change to type: NodePort. The targetPort field is still required, as NodePorts are backed by a ClusterIP service.

Applying the above manifest will assign your NodePort a random port number from the range available to Kubernetes. This usually defaults to ports 30000-32767. You can manually specify a port by setting the ports.nodePort field:

spec:
  ports:

-   name: http
    port: 80
    targetPort: 80
    nodePort: 32000
    protocol: TCP

This will route traffic on port 32000 to port 80 in your Pods.

NodePorts aren’t often ideal for public services. They use non-standard ports, which are unsuitable for most HTTP traffic. You can use a NodePort to quickly set up a service for development use or to expose a TCP or UDP service on its own port. When serving a production environment to users, you’ll want to use an alternative instead.
