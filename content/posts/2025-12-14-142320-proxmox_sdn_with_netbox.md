---
title: "Proxmox SDN with NetBox"
date: 2025-12-14
draft: false
---

## Pre-requisite {#pre-requisite}

-   [Proxmox VE (PVE)]({{< relref "20230228043925-proxmox_ve.md" >}})
-   [NetBox: The Network Source of Truth]({{< relref "2025-11-30-174834-netbox.md" >}})
-   [NetBox Installation]({{< relref "2025-12-14-141844-netbox_installation.md" >}})
-   [Proxmox SDN]({{< relref "2025-12-14-142616-proxmox_sdn.md" >}})


## Create API Token {#create-api-token}

To allow Proxmox to talk to NetBox, you need an API token.

1.  Log in to NetBox.
2.  Click on your **Username** (top right) -&gt; **API Tokens**.
3.  Click **Add a Token**.
4.  **Description**: \`Proxmox SDN\`
5.  **Write Enabled**: Checked (Proxmox needs to write IP data).
6.  Click **Create**.
7.  **Copy the Token**. You will need this for the Proxmox SDN Controller setup.


## Configure Proxmox SDN with NetBox {#configure-proxmox-sdn-with-netbox}

Now that NetBox is running, connect Proxmox to it.


### Add NetBox IPAM Plugin {#add-netbox-ipam-plugin}

1.  Go to **Datacenter -&gt; SDN -&gt; Options -&gt; IPAM**.
2.  Click **Add -&gt; NetBox**.
3.  **ID**: **netbox**
4.  **URL**: **<http://192.168.1.24/api>** (Your NetBox URL; no trailing slash or /api path).
5.  **Token**: Paste the API Token you generated.
6.  Click **Add**.


### Important: Pre-create Prefixes in NetBox {#important-pre-create-prefixes-in-netbox}

Proxmox **will not** create the subnet/prefix in NetBox for you. You must do it manually first.

1.  In NetBox, go to **IPAM -&gt; Prefixes -&gt; Add**.
2.  Add the prefix that matches your Proxmox VNet subnet (e.g., \`10.60.10.0/24\`).
3.  Set Status to **Active**.


### Assign NetBox to a Zone {#assign-netbox-to-a-zone}

1.  Go to **Datacenter -&gt; SDN -&gt; Zones**.
2.  Edit your EVPN Zone (e.g., \`evpntest\`).
3.  Change **IPAM** to **netbox**.
4.  Click **OK**.


### Apply &amp; Verify {#apply-and-verify}

1.  In Proxmox, go to **Datacenter -&gt; SDN -&gt; Apply**.
2.  Restart a VM/LXC in that VNet.
3.  Check NetBox IPAM; you should see the IP address automatically registered with the VM's name.
