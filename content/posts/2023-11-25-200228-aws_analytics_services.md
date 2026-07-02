---
title: "AWS Analytics services"
draft: false
---

The are several AWS Analytics services and these include:

-   [Amazon Athena]({{< relref "2023-11-25-200445-amazon_athena.md" >}})
-   [Amazon EMR]({{< relref "2023-11-04-230959-amazon_emr.md" >}})
-   Amazon CloudSearch
-   Amazon Elasticsearch Service
-   [Amazon Kinesis]({{< relref "2023-11-25-203437-amazon_kinesis.md" >}})
-   Amazon QuickSight
-   Amazon Data Pipeline
-   [AWS Glue]({{< relref "2023-11-25-202232-aws_glue.md" >}})
-   AWS Lake Formation
-   Amazon MSK


## Data Analysis And Query Use Cases {#data-analysis-and-query-use-cases}

Query services like Amazon Athena, data warehouses like Amazon Redshift, and sophisticated data processing frameworks like Amazon EMR, all address different needs and use cases.

| AWS Service                                                              | Primary Use Case | When to use                                                                                                                                                                                                                                |
|--------------------------------------------------------------------------|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Amazon Athena]({{< relref "2023-11-25-200445-amazon_athena.md" >}})     | Query            | Run interactive queries against data directly in Amazon S3 without worrying about formatting data or managing infrastructure. Can use with other services such as [Amazon Redshift]({{< relref "2023-11-04-225621-amazon_redshift.md" >}}) |
| [Amazon Redshift]({{< relref "2023-11-04-225621-amazon_redshift.md" >}}) | Data warehouse   | Pull data from many sources, format and organize it, store it, and support complex, high speed queries that produce business reports.                                                                                                      |
| [Amazon EMR]({{< relref "2023-11-04-230959-amazon_emr.md" >}})           | Data Processing  | Highly distributed processing frameworks such as Hadoop, Spark, and Presto. Run a wide variety of scale-out data processing tasks for applications such as machine learning, graph analytics, data transformation, streaming data.         |
| [AWS Glue]({{< relref "2023-11-25-202232-aws_glue.md" >}})               | ETL Service      | Transform and move data to various destinations. Used to prepare and load data for analytics. Data source can be S3, RedShift or another database. Glue Data Catalog can be queried by Athena, EMR and RedShift Spectrum                   |
