---
title: "Helm chart repository"
draft: false
---

## What is a chart repository? {#what-is-a-chart-repository}

A chart repository is an HTTP server that houses packaged charts and an index.yaml file. That file has an index of all the charts in the repository. A chart repository can be any HTTP server that can serve YAML and .tar files and can answer GET requests. Therefore, you have many options for hosting your chart repository. You can use a Google Cloud Storage bucket, an Amazon S3 bucket, GitHub pages, or you can create a web server.


### Commands for working with repositories {#commands-for-working-with-repositories}

```console
$ helm repo list

NAME      URL
stable    https://kubernetes-charts.storage.googleapis.com/
$ helm search jenkins

NAME                VERSION       DESCRIPTION
stable/jenkins      0.1.14        A Jenkins Helm chart for Kubernetes.
$ helm repo add my-charts https://my-charts.storage.googleapis.com

$ helm repo list

NAME           URL
stable        https://kubernetes-charts.storage.googleapis.com/
my-charts     https://my-charts.storage.googleapis.com
```
