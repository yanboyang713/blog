---
title: "stream processing framework"
draft: false
---

Data Stream Processing Engines (DSPEs) lie at the core of DSPSs and enable the definition and execution of stream processing pipelines.


## Use case {#use-case}

Under several application scenarios such as

-   fraud detection in financial transactions
-   healthcare analytics involving digital sensors
-   [Internet of Things (IoT)]({{< relref "2023-10-09-180602-industrial_internet_of_things_iiot.md" >}})


## MOTIVATION {#motivation}

Modern Data Stream Processing Systems (DSPS) try to combine batch and stream processing capabilities into a single or multiple parallel data processing pipelines.


## ARCHITECTURE OF A DSPS {#architecture-of-a-dsps}

the architecture of a DSPS is generally multi-tiered and is composed of many loosely coupled components that include data sources, data collection systems, data storage systems, messaging systems, and stream processing and delivery systems.
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1697167946/big%20data/r3gnrdxyifqvd2aci0dc.png)


### data stream ingestion layer {#data-stream-ingestion-layer}

Data ingestion is the process of getting data streams from its source to its processing or [storage]({{< relref "20230331193459-storage.md" >}}) system.

There are many sources of input data streams [28]. These include data streams from various IoT devices such as sensors, video and other electronic monitors, social network Application Programming Interfaces (APIs), [WebSockets]({{< relref "2023-10-12-234245-websockets.md" >}}), Representational State Transfer ([RESTful]({{< relref "20230620161819-restful.md" >}})) Web services, service usage logs, other stream processing systems, or any object which can collect and transmit time-sensitive data.

Queueing systems encompass the spectrum of messaging services, from the traditional message queuing products such as MQTT, RabbitMQ, and ActiveMQ to the newer products such as NSQ and ZeroMQ [9], [29]. Apache [Kafka]({{< relref "2023-10-26-032050-kafka.md" >}}) and DistributedLog have grown to embody more than a message system, and both currently support publishing and subscribing to streams of records [8]. There are also many commercial stream ingestion systems including Scribe [26] developed at Facebook, Kinesis Data Firehose managed by Amazon Web Services (AWS), IBM WebSphere MQ and [Messaging services on Azure]({{< relref "2023-10-12-234611-messaging_services_on_azure.md" >}}) [29].
[Message Queue]({{< relref "2023-10-28-182217-message_queue.md" >}})


### data stream processing layer {#data-stream-processing-layer}

The data stream processing layer is where the streaming data processing applications or jobs are executed. It can host loosely coupled disjoint applications or a DSPE or both. DSPEs generally offer a set of streaming data processing operators which can be configured and threaded together to build a stream data processing pipeline to analyze incoming data streams [30].

[data stream processing engines (DSPEs)]({{< relref "2023-10-13-000706-data_stream_processing_engines_dspes.md" >}})


### [storage]({{< relref "20230331193459-storage.md" >}}) layer {#storage--20230331193459-storage-dot-md--layer}

DSPSs often store analyzed data, discovered patterns and extracted knowledge from different data processing stages for further processing.

DSPS architecture ranges from traditional file systems such as HDFS and Baidu File System (BFS) to distributed file relational databases such as [PostgreSQL]({{< relref "20230220215653-postgre.md" >}}), key-value stores such as [Redis]({{< relref "2023-10-12-235342-redis.md" >}}), in-memory databases such as VoltDB, document storage such as MongoDB, graph storage systems such as Neo4j, NoSQL databases such as Cassandra, and NewSQL such as CockroachDB [38].

[Azure Data Lake]({{< relref "20230104141434-azure_data_lake.md" >}})


### resource management layer {#resource-management-layer}

The resource management layer coordinates actions among compute and storage nodes and manages resource allocation and scheduling in distributed systems to enable parallel processing of high volume and velocity of data streams [39].

[Kubernetes]({{< relref "20230105185343-kubernetes.md" >}})


### Data Stream Output Layer {#data-stream-output-layer}

The results from [data stream processing](#data-stream-processing-layer) pipelines can be directed to an application, another workflow, a [data visualization]({{< relref "2023-10-25-002700-data_visualization.md" >}}) tool, or an alert or monitoring dashboard [8].

[Prometheus]({{< relref "20230602142910-prometheus.md" >}})


## Reference List {#reference-list}

1.  Isah, H., Abughofa, T., Mahfuz, S., Ajerla, D., Zulkernine, F., &amp; Khan, S. (2019). A survey of distributed data stream processing frameworks. IEEE Access, 7, 154300-154316.
