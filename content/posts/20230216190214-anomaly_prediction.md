---
title: "Wireless Anomaly detection Project"
draft: false
---

## Metrics {#metrics}

Physical Layer:

\begin{itemize}
    \item H-Phy-RX: RX packets information at the physical layer
    \item H-Phy-TX: TX packets information at the physical layer
    \item H-Phy-RX-Drop: RX drop packets information at the physical layer
    \item H-Phy-TX-Drop: TX drop packets information at the physical layer
\end{itemize}

Mac Layer:

\begin{itemize}
    \item H-Mac-Dequeue: Dequeue packets information at the Mac layer
    \item H-Mac-Enqueue: Enqueue packets information at the Mac layer
    \item H-Mac-Queue-Drop: Drop packets information at the Mac layer
    \item H-Mac-Before-Enqueue-Drop: Drop packets information before enqueue at the Mac layer
    \item H-Mac-After-Dequeue-Drop: Drop packets information after dequeue at the Mac layer
\end{itemize}

H-Phy-RX:

\begin{itemize}
    \item Packet information with timestamp
    \item Signal Dbm
    \item Noise Dbm
\end{itemize}

H-Phy-TX:

\begin{itemize}
    \item Packet information with timestamp
    \item Power Level
\end{itemize}

H-Phy-RX-Drop:

\begin{itemize}
    \item Packet information with timestamp
    \item Failure Reason
\end{itemize}

H-Phy-TX-Drop:

\begin{itemize}
    \item Packet information with timestamp
\end{itemize}

H-Mac-Dequeue:

\begin{itemize}
    \item Packet information with timestamp
\end{itemize}

H-Mac-Enqueue:

\begin{itemize}
    \item Packet information with timestamp
\end{itemize}

H-Mac-Queue-Drop:

\begin{itemize}
    \item Packet information with timestamp
\end{itemize}

H-Mac-Before-Enqueue-Drop:

\begin{itemize}
    \item Packet information with timestamp
\end{itemize}

H-Mac-After-Dequeue-Drop:

\begin{itemize}
    \item Packet information with timestamp
\end{itemize}

Totally, have 13 measurement metrics.


## Database Set-up {#database-set-up}

[TimeScaleDB]({{< relref "20230216045729-timescaledb.md" >}})

```sql
SELECT datname FROM pg_catalog.pg_database;

CREATE DATABASE anomalyprediction;
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
```

```sql
--Anomaly Prediction Project

--runrecord
DROP TABLE runrecord;
CREATE TABLE runrecord (
        "runid" TEXT,
        "hostname" TEXT,
        "datamode" TEXT,
        "controlmode" TEXT,
        "numofnodes" INTEGER,
        "simulationtime" REAL,
        "payloadsize" INTEGER,
        "datarate" TEXT,
        "channelnumber" INTEGER,
        "frequency" REAL,
        "standard" TEXT,
        "channelwidth" INTEGER,
        "guardinterval" INTEGER,
        PRIMARY KEY (runid, hostname)
);
--rxpacketinfo
DROP TABLE rxpacketinfo;
CREATE TABLE rxpacketinfo (
        "runid" TEXT,
        "hostname" TEXT,
        "type" TEXT,
        "time" TIMESTAMP WITHOUT TIME ZONE NOT NULL,
        "context" TEXT,
        "nodename" TEXT,
        "nodeid" INTEGER,
        "signaldb" REAL,
        "noisedb" REAL,
        "channelfreqmhz" INTEGER,
        "ness" INTEGER,
        "nss" INTEGER,
        "powerlevel" INTEGER,
        "packetsize" INTEGER,
        "packetuid" INTEGER,
        PRIMARY KEY (runid, hostname, packetuid, context, time),
    FOREIGN KEY (runid, hostname)
                REFERENCES runrecord (runid, hostname)
        ON DELETE CASCADE
        ON UPDATE NO ACTION
);

SELECT create_hypertable('rxpacketinfo','time');
CREATE INDEX ON rxpacketinfo (packetuid, time DESC);

--txpacketinfo
DROP TABLE txpacketinfo;
CREATE TABLE txpacketinfo (
        "runid" TEXT,
        "hostname" TEXT,
        "type" TEXT,
        "time" TIMESTAMP WITHOUT TIME ZONE NOT NULL,
        "context" TEXT,
        "nodename" TEXT,
        "nodeid" INTEGER,
        "channelfreqmhz" INTEGER,
        "ness" INTEGER,
        "nss" INTEGER,
        "powerlevel" INTEGER,
        "packetsize" INTEGER,
        "packetuid" INTEGER,
        PRIMARY KEY (runid, hostname, packetuid, context, time),
    FOREIGN KEY (runid, hostname)
                REFERENCES runrecord (runid, hostname)
        ON DELETE CASCADE
        ON UPDATE NO ACTION
);

SELECT create_hypertable('txpacketinfo','time');
CREATE INDEX ON rxpacketinfo (packetuid, time DESC);

--phytxdropinfo
DROP TABLE phytxdropinfo;
CREATE TABLE phytxdropinfo (
        "runid" TEXT,
        "hostname" TEXT,
        "type" TEXT,
        "time" TIMESTAMP WITHOUT TIME ZONE NOT NULL,
        "context" TEXT,
        "nodename" TEXT,
        "nodeid" INTEGER,
        "packetsize" INTEGER,
        "packetuid" INTEGER,
        PRIMARY KEY (runid, hostname, packetuid, context, time),
    FOREIGN KEY (runid, hostname)
                REFERENCES runrecord (runid, hostname)
        ON DELETE CASCADE
        ON UPDATE NO ACTION
);

SELECT create_hypertable('phytxdropinfo','time');
CREATE INDEX ON rxpacketinfo (packetuid, time DESC);

--phytxdropinfo
DROP TABLE phyrxdropinfo;
CREATE TABLE phyrxdropinfo (
        "runid" TEXT,
        "hostname" TEXT,
        "type" TEXT,
        "time" TIMESTAMP WITHOUT TIME ZONE NOT NULL,
        "context" TEXT,
        "nodename" TEXT,
        "nodeid" INTEGER,
        "packetsize" INTEGER,
        "packetuid" INTEGER,
        "failurereason" TEXT,
        PRIMARY KEY (runid, hostname, packetuid, context, time),
    FOREIGN KEY (runid, hostname)
                REFERENCES runrecord (runid, hostname)
        ON DELETE CASCADE
        ON UPDATE NO ACTION
);

SELECT create_hypertable('phyrxdropinfo','time');
CREATE INDEX ON rxpacketinfo (packetuid, time DESC);

--queuedropinfo
DROP TABLE queuedropinfo;
CREATE TABLE queuedropinfo (
        "runid" TEXT,
        "hostname" TEXT,
        "type" TEXT,
        "time" TIMESTAMP WITHOUT TIME ZONE NOT NULL,
        "context" TEXT,
        "nodename" TEXT,
        "nodeid" INTEGER,
        "packetsize" INTEGER,
        "packetuid" INTEGER,
        PRIMARY KEY (runid, hostname, packetuid, context, time),
    FOREIGN KEY (runid, hostname)
                REFERENCES runrecord (runid, hostname)
        ON DELETE CASCADE
        ON UPDATE NO ACTION
);

```


## Distribution {#distribution}

[Nikagumi Rice Distribution]({{< relref "20230304092420-nikagumi_rice_distribution.md" >}})


## [c++ runID]({{< relref "20230307105422-c_runid.md" >}}) {#c-plus-plus-runid--20230307105422-c-runid-dot-md}


## [Jupyter]({{< relref "20230307124643-jupyter.md" >}}) {#jupyter--20230307124643-jupyter-dot-md}

```bash
conda create -n anomaly python=3.7.7
conda activate anomaly
conda install notebook ipykernel
ipython kernel install --user --name anomaly --display-name "Python (anomaly)"
conda deactivate
```


## Papers {#papers}

-   [Deep Learning for multivariate time series data Anomaly Detection]({{< relref "20230103080850-deep_learning_for_multivariate_time_series_data_anomaly_detection.md" >}})


## Reference List {#reference-list}

1.
