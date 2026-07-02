---
title: "TimeScaleDB Installation"
draft: false
---

[TimeScaleDB]({{< relref "20230216045729-timescaledb.md" >}})


## Single Node in Docker {#single-node-in-docker}


### Pre-installation {#pre-installation}

-   [docker]({{< relref "20230216042646-docker.md" >}})


### Install {#install}

```bash
docker run -d --name timescaledb -p 5432:5432 \
-e POSTGRES_PASSWORD=password timescale/timescaledb:latest-pg14

docker start /timescaledb
docker container ls
docker ps -a
```

```file
User name:postgres
Password: password
```


### Remove {#remove}

```bash
docker rm /timescaledb
```


## Muilt-Node {#muilt-node}

[TimescaleDB on Kubernetes]({{< relref "20230419011320-timescaledb_on_kubernetes.md" >}})
