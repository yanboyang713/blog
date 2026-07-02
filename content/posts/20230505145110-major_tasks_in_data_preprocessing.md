---
title: "Major Tasks in Data Preprocessing"
draft: false
---

We will define 5 major tasks for data preprocessing framework, namely: **Get**, **Understand**, **Tidy** &amp; **Manipulate**, **Scan** and **Transform**.

A typical [data preprocessing]({{< relref "20230505144809-data_preprocessing.md" >}}) process usually (but not necessarily) follows the following order of tasks given below:

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1615144175/MATH2349/DataPreprocessing_whuabu.png" >}}

1.  **Get**: A data set can be stored in a computer or can be online in different file formats. We need to get the data set into R by importing it from other data sources (i.e., .txt, .xls, .csv files and databases) or scraping from web. R provides many useful commands to import (and export) data sets in different file formats.
2.  **Understand**: We cannot perform any type of data preprocessing without understanding what we have in hand. In this step, we will check the data volume (i.e., the dimensions of the data) and structure, understand the variables/attributes in the data set, and understand the meaning of each level/value for the variables.
3.  **Tidy &amp; Manipulate**: In this step, we will apply several important tasks to tidy up messy data sets. We will follow Hadley Wickham’s “Tidy Data” principles:
    -   Each variable must have its own column.
    -   Each observation must have its own row.
    -   Each value must have its own cell.
    -   We may also need to manipulate, i.e. filter, arrange, select, subset/split data, or generate new variables from the data set.
4.  **Scan**: This step will include checking for plausibility of values, cleaning data for obvious errors, identifying and handling outliers, and dealing with missing values.
5.  **Transform**: Some statistical analysis methods are sensitive to the scale of the variables and it may be necessary to apply transformations before using them. In this step we will introduce well-known data transformations, data scaling, centering, standardising and normalising methods.

There are also other steps related with preprocessing special types of data including dates, time and characters/strings. The last module of this course will introduce the special operations used for date, time and text preprocessing.
