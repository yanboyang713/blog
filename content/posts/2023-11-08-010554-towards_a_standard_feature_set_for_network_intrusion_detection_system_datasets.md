---
title: "Network Intrusion Detection System Datasets"
draft: false
---

evaluates standard [Network Intrusion Detection Systems (NIDSs)]({{< relref "2023-11-08-034959-network_intrusion_detection_systems_nidss.md" >}}) feature sets based on the [NetFlow]({{< relref "2023-11-08-035705-netflow.md" >}}) network meta-data collection protocol and system.


## Datasets {#datasets}

Where the UNSW-NB15 and CSE-CIC-IDS2018 datasets have very high benign-toattack ratios, whereas the ToN-IoT and BoT-IoT datasets are mainly made up of attack samples, which do not represent a realistic network behaviour.
some of the features in the UNSW-NB15, BoT-IoT, and CSE-CIC-IDS2018 datasets are handcrafted features that are not originally found in network packets but are statistically calculated based on other features, such as the total number of bytes transferred over the last 100 seconds.

-   [UNSW-NB15]({{< relref "2023-11-08-010913-unsw_nb15.md" >}})
-   [BoT-IoT]({{< relref "2023-11-08-011045-bot_iot.md" >}})
-   [ToN-IoT]({{< relref "2023-11-08-011250-ton_iot.md" >}})
-   [CICIDS2017 dataset]({{< relref "2023-11-08-005255-cicids2017_dataset.md" >}})

<a id="figure--fig:SED-HR4049"></a>

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699424660/networking%20datasets/Towards-a-Standard-Feature-Set-for-Network-Intrusion-Detection-System-Datasets_sc3srk.png" caption="<span class=\"figure-number\">Figure 1: </span>Networking datasets" width="300px" >}}


### NetFlow features {#netflow-features}

If a data flow is located in the attack events it would be labelled as an attack (class 1) in the binary label and its respective attack’s type would be recorded in the attack label, otherwise, the sample is labelled as a benign flow (class 0).


### [Data Preprocessing]({{< relref "20230505144809-data_preprocessing.md" >}}) {#data-preprocessing--20230505144809-data-preprocessing-dot-md}

As part of the data pre-processing, the flow identifiers such as IDs, source/destination IP and ports, timestamps, and start/end time are dropped to avoid learning bias towards attacking and victim end nodes. For the UNSW-NB15 and NF-UNSW-NB15-v2 datasets, The Time To Live (TTL)-based features are dropped due to their extreme correlation with the labels. Additionally, the minmax normalisation technique has been applied to scale all datasets’ values between 0 and the datasets have been split into 70%-30% for training and testing purposes.


### Binary-class classification {#binary-class-classification}

Extra Tree classifier to compare the predictive power of our proposed NetFlow based feature set, with the proprietary features sets provided with the original benchmark NIDS datasets.
[Extremely Randomized Trees Classifier(Extra Trees Classifier)]({{< relref "2023-11-08-010708-extremely_randomized_trees_classifier_extra_trees_classifier.md" >}})


### Multi-class classification {#multi-class-classification}


## Reference List {#reference-list}

1.  <https://link.springer.com/article/10.1007/s11036-021-01843-0>
2.  <https://staff.itee.uq.edu.au/marius/NIDS_datasets/>
3.  Sarhan, M., Layeghy, S., &amp; Portmann, M. (2022). Towards a standard feature set for network intrusion detection system datasets. Mobile networks and applications, 1-14.
