---
title: "create proxmox cluster"
draft: false
---

## Create a Cluster {#create-a-cluster}

You can either create a cluster on the console (login via ssh), or through the API using the Proxmox VE web interface (Datacenter → Cluster).
**Note:** Use a unique name for your cluster. This name cannot be changed later. The cluster name follows the same rules as node names.


### Create via Web GUI {#create-via-web-gui}

Under Datacenter → Cluster -&gt; Create Cluster, click on Create Cluster. Enter the cluster name and select a network connection from the drop-down list to serve as the main cluster network (Link 0). It defaults to the IP resolved via the node’s hostname.

**Note:** Ensure that the network selected for cluster communication is not used for any high traffic purposes, like network storage or live-migration. While the cluster network itself produces small amounts of data, it is very sensitive to latency.


### Create via the Command Line {#create-via-the-command-line}

Login via ssh to the first Proxmox VE node and run the following command:

```bash
hp1# pvecm create CLUSTERNAME
```

To check the state of the new cluster use:

```bash
hp1# pvecm status
```


## Adding Nodes to the Cluster {#adding-nodes-to-the-cluster}


### Join Node to Cluster via GUI {#join-node-to-cluster-via-gui}

1.  Log in to the web interface on an existing cluster node. Under Datacenter → Cluster, click the Join Information button at the top. Then, click on the button Copy Information. Alternatively, copy the string from the Information field manually.
2.  Next, log in to the web interface on the node you want to add. Under Datacenter → Cluster, click on Join Cluster. Fill in the Information field with the Join Information text you copied earlier. Most settings required for joining the cluster will be filled out automatically. For security reasons, the cluster password has to be entered manually.

**Note:** To enter all required data manually, you can disable the Assisted Join checkbox.

After clicking the Join button, the cluster join process will start immediately. After the node has joined the cluster, its current node certificate will be replaced by one signed from the cluster certificate authority (CA). This means that the current session will stop working after a few seconds. You then might need to force-reload the web interface and log in again with the cluster credentials.

Now your node should be visible under Datacenter → Cluster.


### Join Node to Cluster via Command Line {#join-node-to-cluster-via-command-line}

Log in to the node you want to join into an existing cluster via ssh.

For IP-ADDRESS-CLUSTER, use the IP or hostname of an existing cluster node. An IP address is recommended (see Link Address Types).

To check the state of the cluster use:

```bash

 # pvecm status
Cluster status after adding 4 nodes
 # pvecm status
Cluster information
~~~~~~~~~~~~~~~~~~~
Name:             prod-central
Config Version:   3
Transport:        knet
Secure auth:      on

Quorum information
~~~~~~~~~~~~~~~~~~
Date:             Tue Sep 14 11:06:47 2021
Quorum provider:  corosync_votequorum
Nodes:            4
Node ID:          0x00000001
Ring ID:          1.1a8
Quorate:          Yes

Votequorum information
~~~~~~~~~~~~~~~~~~~~~~
Expected votes:   4
Highest expected: 4
Total votes:      4
Quorum:           3
Flags:            Quorate

Membership information
~~~~~~~~~~~~~~~~~~~~~~
    Nodeid      Votes Name
0x00000001          1 192.168.15.91
0x00000002          1 192.168.15.92 (local)
0x00000003          1 192.168.15.93
0x00000004          1 192.168.15.94
If you only want a list of all nodes, use:

 # pvecm nodes
List nodes in a cluster
 # pvecm nodes

Membership information
~~~~~~~~~~~~~~~~~~~~~~
    Nodeid      Votes Name
         1          1 hp1
         2          1 hp2 (local)
         3          1 hp3
         4          1 hp4

```


## Remove a Cluster Node {#remove-a-cluster-node}

Caution	Read the procedure carefully before proceeding, as it may not be what you want or need.
Move all virtual machines from the node. Ensure that you have made copies of any local data or backups that you want to keep. In addition, make sure to remove any scheduled replication jobs to the node to be removed.

Caution	Failure to remove replication jobs to a node before removing said node will result in the replication job becoming irremovable. Especially note that replication automatically switches direction if a replicated VM is migrated, so by migrating a replicated VM from a node to be deleted, replication jobs will be set up to that node automatically.
In the following example, we will remove the node hp4 from the cluster.

Log in to a different cluster node (not hp4), and issue a pvecm nodes command to identify the node ID to remove:

```bash
 hp1# pvecm nodes

Membership information
~~~~~~~~~~~~~~~~~~~~~~
    Nodeid      Votes Name
         1          1 hp1 (local)
         2          1 hp2
         3          1 hp3
         4          1 hp4
```

At this point, you must power off hp4 and ensure that it will not power on again (in the network) with its current configuration.

Important	As mentioned above, it is critical to power off the node before removal, and make sure that it will not power on again (in the existing cluster network) with its current configuration. If you power on the node as it is, the cluster could end up broken, and it could be difficult to restore it to a functioning state.
After powering off the node hp4, we can safely remove it from the cluster.

 hp1# pvecm delnode hp4
 Killing node 4
Note	At this point, it is possible that you will receive an error message stating Could not kill node (error = CS_ERR_NOT_EXIST). This does not signify an actual failure in the deletion of the node, but rather a failure in corosync trying to kill an offline node. Thus, it can be safely ignored.
Use pvecm nodes or pvecm status to check the node list again. It should look something like:

hp1# pvecm status

...

Votequorum information
`~~~~~~~~~~~~~~~~~~~~`
Expected votes:   3
Highest expected: 3
Total votes:      3
Quorum:           2
Flags:            Quorate

Membership information
`~~~~~~~~~~~~~~~~~~~~`
    Nodeid      Votes Name
0x00000001          1 192.168.15.90 (local)
0x00000002          1 192.168.15.91
0x00000003          1 192.168.15.92
If, for whatever reason, you want this server to join the same cluster again, you have to:

do a fresh install of Proxmox VE on it,

then join it, as explained in the previous section.

Note	After removal of the node, its SSH fingerprint will still reside in the known_hosts of the other nodes. If you receive an SSH error after rejoining a node with the same IP or hostname, run pvecm updatecerts once on the re-added node to update its fingerprint cluster wide.


## Reference List {#reference-list}

1.  <https://pve.proxmox.com/wiki/Cluster_Manager>
