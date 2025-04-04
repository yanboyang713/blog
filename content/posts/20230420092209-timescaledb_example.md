---
title: "TimescaleDB Example"
draft: false
---

## Create Database {#create-database}

```sql
CREATE DATABASE vicroads;
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
```


## Create Table {#create-table}

```sql
DROP TABLE detector;
CREATE TABLE detector (
        "id" INTEGER PRIMARY KEY,
        "name" TEXT,
        "link_key" TEXT,
        "description" TEXT,
        "x" NUMERIC,
        "y" NUMERIC
);
```
