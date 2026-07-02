---
title: "Proxmox NetBox Sync"
date: 2025-12-14
draft: false
---

[Proxmox VE (PVE)]({{< relref "20230228043925-proxmox_ve.md" >}})
[NetBox: The Network Source of Truth]({{< relref "2025-11-30-174834-netbox.md" >}})


## Pre-requires {#pre-requires}

-   [NetBox Installation]({{< relref "2025-12-14-141844-netbox_installation.md" >}})


## net-pve-sync {#net-pve-sync}

`net-pve-sync` (based on `netbox-pve-sync`) is a pull-based synchronization job that reads VM/inventory data from the
Proxmox VE API and reconciles it into NetBox (create/update/delete as needed).

-   Sync direction: Proxmox VE -&gt; NetBox (VMs, disks, interfaces, IPs/prefixes, MACs, VLANs, and a few metadata fields).
-   Auth: NetBox API token + Proxmox API token (read-only perms like `VM.Monitor`, `VM.Audit`, `Sys.Audit` are typically sufficient).
-   NetBox prep: create the cluster and physical hosts, and add the custom fields expected by the sync (ex: `autostart`, `ha`, `backup`).


### Prepare NetBox {#prepare-netbox}


#### Create a test Site {#create-a-test-site}

-   Left sidebar → Organization → Sites
-   Click Add
-   Fill the required fields:
    -   Name: Lab (or Test)
    -   Slug: lab (auto-fills; must be unique)
-   Optional (nice to set):
    -   Status: Active
    -   Description: Test objects for Proxmox sync
-   Click Create


#### (Optional) Create a test Tenant Group {#optional--create-a-test-tenant-group}

-   Left sidebar → Tenancy → Tenant Groups
-   Click Add
-   Set Name: Test, Slug: test
-   Click Create


#### Create a test Tenant {#create-a-test-tenant}

-   Left sidebar → Tenancy → Tenants
-   Click Add
-   Fill:
    -   Name: Lab Tenant
    -   Slug: lab-tenant
    -   Group: select Test (if you created one)
-   Click Create


#### Create the minimum Device metadata (one-time) {#create-the-minimum-device-metadata--one-time}

-   Devices → Devices TYPES → Manufacturers → Add → Name: Generic → Create
-   Devices → Devices TYPE → Device Types → Add → Manufacturer: Generic, Model: Proxmox Node → Create
-   Devices → Device Roles → Add → Name: Hypervisor, Color: any → Create


#### Create the [NetBox]({{< relref "2025-11-30-174834-netbox.md" >}}) Cluster that will hold the VMs {#create-the-netbox--2025-11-30-174834-netbox-dot-md--cluster-that-will-hold-the-vms}

-   Virtualization → Cluster Types → Add → Name: Proxmox → Create
-   (Optional) Virtualization → Cluster Groups → Add → Name: Lab → Create
-   Virtualization → Clusters → Add
    -   Name: Boyang (or Boyang-Lab)
    -   Type: Proxmox
    -   Group: Lab (optional)
    -   Create
-   Open the “Boyang” cluster in your browser and look at the address bar; it will be like:

    -   <https://<netbox>/virtualization/clusters/123/>

    In that case, **NB_CLUSTER_ID=123**. My one is 1.


#### Create the Proxmox nodes as Devices (must match names) {#create-the-proxmox-nodes-as-devices--must-match-names}

first: Go to Devices → Devices → Configure Table → Add **Cluster** column
Then:

-   Devices → Devices → Add
    -   Name: server1 (repeat for server2…server5)
    -   Site: your test Site
    -   Device Role: Hypervisor
    -   Device Type: Proxmox Node
    -   Status: Active
    -   Cluster: [Boyang](#create-the-netbox--2025-11-30-174834-netbox-dot-md--cluster-that-will-hold-the-vms)
-   Important: the device name must match Proxmox node name (server1, etc.) or the sync exits.


#### Add the Custom Fields the tool requires {#add-the-custom-fields-the-tool-requires}

-   Customization → Custom Fields → Add (create each)
    -   Name autostart, Type Boolean, Object types: Virtualization &gt; Virtual Machine, Label Autostart
    -   Name replicated, Type Boolean, Object types: Virtualization &gt; Virtual Machine, Label Replicated
    -   Name ha, Type Boolean, Object types: Virtualization &gt; Virtual Machine, Label Failover
    -   Name backup, Type Boolean, Object types: Virtualization &gt; Virtual Disk, Label Backup
    -   Name **dns_name**, Type Text, Object types: IPAM &gt; Prefix, Label DNS Name

**Proxmox-side note** (for IP sync)

-   IPs come from QEMU Guest Agent (network-get-interfaces). If your VMs don’t have it installed/enabled, VMs will still sync, but IPs/prefixes likely won’t.


#### Create [NetBox]({{< relref "2025-11-30-174834-netbox.md" >}}) API Token {#create-netbox--2025-11-30-174834-netbox-dot-md--api-token}

Create **NB_API_TOKEN** (NetBox 4.4.9)

-   Top right: click your username → Profile (or User Profile)
-   Go to API Tokens tab/section → Add / Create Token
-   Set a name like netbox-pve-sync (optional: set an expiry) → Create
-   Copy the token value shown (you won’t be able to see it again) → this is **NB_API_TOKEN**

If you don’t see API Tokens, your account lacks permission; use an admin account or have an admin grant API token creation.

Your **NB_API_URL**

-   It’s the base URL of NetBox: **NB_API_URL=<https://192.168.1.26>**
-   (Optional quick check) <https://192.168.1.26/api/> should load the API root in a browser.

<!--list-separator-->

-  Verify token works

    ```bash
    curl -k -H "Authorization: Token <NETBOX_TOKEN>" https://192.168.1.26/api/
    ```


### Prepare Proxmox (API access with least privilege) {#prepare-proxmox--api-access-with-least-privilege}


#### Create a dedicated user for [Proxmox VE (PVE)]({{< relref "20230228043925-proxmox_ve.md" >}}) and [NetBox]({{< relref "2025-11-30-174834-netbox.md" >}}) sync {#create-a-dedicated-user-for-proxmox-ve--pve----20230228043925-proxmox-ve-dot-md--and-netbox--2025-11-30-174834-netbox-dot-md--sync}

-   Proxmox UI → Datacenter → Permissions → Users → Add
-   Fill:
    -   User name: netsync
    -   Realm: pve
    -   Password: set a strong password (only needed to manage the user; the sync will use a token)
-   Add


#### Give netsync dedicated user Permissions {#give-netsync-dedicated-user-permissions}

-   Datacenter → Permissions → Add → User Permission
-   Path: /
-   User: netsync@pve
-   Role: PVEAuditor
-   Propagate: checked
-   Add


#### Create API Tokens for [Proxmox VE (PVE)]({{< relref "20230228043925-proxmox_ve.md" >}}) and [NetBox]({{< relref "2025-11-30-174834-netbox.md" >}}) sync {#create-api-tokens-for-proxmox-ve--pve----20230228043925-proxmox-ve-dot-md--and-netbox--2025-11-30-174834-netbox-dot-md--sync}

-   Datacenter → Permissions → API Tokens → Add
-   User: netsync@pve
-   Token ID: netbox-sync
-   Enable Privilege Separation
-   Add, copy the secret (**PVE_API_SECRET**)


#### Add API Token Permissions {#add-api-token-permissions}

-   Datacenter → Permissions → Add (API Token Permission)
-   Path: /
-   API Token: netsync@pve!netbox-sync
-   Role:  PVEAuditor
-   Propagate: checked
-   Add


#### Verify {#verify}

<!--list-separator-->

-  List VM

    ```bash
    curl -k -i -sS \
      -H 'Authorization: PVEAPIToken=netsync@pve!netbox-sync=<APItoken>' \
      'https://192.168.1.11:8006/api2/json/cluster/resources?type=vm'
    ```

    ```console
    yanboyang713@yanboyang713-Standard-PC-i440FX-PIIX-1996:~$ curl -k -i -sS -H 'Authorization: PVEAPIToken=netsync@pve!netbox-sync=<APItoken>' 'https://192.168.1.11:8006/api2/json/cluster/resources?type=vm'
    HTTP/1.1 200 OK
    Cache-Control: max-age=0
    Connection: Keep-Alive
    Date: Tue, 06 Jan 2026 07:27:22 GMT
    Pragma: no-cache
    Server: pve-api-daemon/3.0
    Content-Length: 6257
    Content-Type: application/json;charset=UTF-8
    Expires: Tue, 06 Jan 2026 07:27:22 GMT

    {"data":[{"memhost":5179975680,"netin":82601058,"maxcpu":8,"maxdisk":34359738368,"diskwrite":1072448512,"name":"jumper","node":"server5","uptime":8825,"vmid":100,"template":0,"tags":"boyang","id":"qemu/100","netout":1060469,"mem":5179975680,"status":"running","maxmem":8589934592,"cpu":0.0202062065371041,"type":"qemu","diskread":1455539712,"disk":0},{"tags":"siyuan","id":"qemu/101","status":"stopped","mem":0,"maxmem":17179869184,"netout":0,"cpu":0,"type":"qemu","diskread":0,"disk":0,"memhost":0,"netin":0,"maxcpu":8,"maxdisk":68719476736,"name":"nss","diskwrite":0,"vmid":101,"node":"server5","uptime":0,"template":1},{"vmid":102,"uptime":0,"node":"server5","template":0,"maxdisk":68719476736,"name":"nss-01","diskwrite":0,"netin":0,"maxcpu":8,"memhost":0,"type":"qemu","diskread":0,"disk":0,"mem":0,"status":"stopped","maxmem":17179869184,"netout":0,"cpu":0,"id":"qemu/102","tags":"siyuan"},{"type":"qemu","disk":0,"diskread":498138112,"netout":664201006,"mem":1003077632,"maxmem":2147483648,"status":"running","cpu":0.00858219526604514,"id":"qemu/103","tags":"boyang","node":"server2","uptime":726803,"vmid":103,"template":0,"maxdisk":8589934592,"diskwrite":2187667456,"name":"Primary-Gateway","netin":712073376,"maxcpu":2,"memhost":2201249792},{"netin":307564775,"maxcpu":1,"memhost":0,"uptime":727151,"node":"server5","vmid":104,"template":0,"maxdisk":10464022528,"diskwrite":248307712,"name":"Primary-DNS","id":"lxc/104","tags":"boyang","type":"lxc","disk":944103424,"diskread":605540352,"netout":319242,"mem":97730560,"maxmem":1073741824,"status":"running","cpu":0},{"cpu":0.0110520107768818,"netout":663749750,"maxmem":2147483648,"mem":977375232,"status":"running","disk":0,"diskread":492862464,"type":"qemu","tags":"boyang","id":"qemu/105","diskwrite":2249975808,"name":"Secondary-Gateway","maxdisk":8589934592,"template":0,"uptime":726916,"node":"server3","vmid":105,"memhost":2200986624,"maxcpu":2,"netin":705417448},{"maxdisk":8350298112,"name":"dhcp","diskwrite":488734720,"vmid":106,"uptime":727117,"node":"server5","template":0,"memhost":0,"netin":307120501,"maxcpu":2,"mem":52088832,"status":"running","maxmem":536870912,"netout":145206,"cpu":5.63781757807018e-05,"type":"lxc","disk":1138196480,"diskread":671559680,"tags":"boyang","id":"lxc/106"},{"diskwrite":228392960,"name":"Secondary-DNS","maxdisk":10464022528,"template":0,"uptime":727040,"node":"server4","vmid":107,"memhost":0,"maxcpu":2,"netin":312698958,"cpu":0,"netout":12469,"mem":66318336,"maxmem":1073741824,"status":"running","disk":944037888,"diskread":565956608,"type":"lxc","tags":"boyang","id":"lxc/107"},{"template":0,"vmid":108,"uptime":437718,"node":"server5","name":"serverless","diskwrite":3207939072,"maxdisk":34359738368,"maxcpu":8,"netin":285982089,"memhost":4362693632,"diskread":2004477440,"disk":0,"type":"qemu","cpu":0.00474642435435331,"mem":3895861248,"status":"running","maxmem":4294967296,"netout":8145308,"id":"qemu/108","tags":"boyang"},{"memhost":0,"maxcpu":1,"netin":307049721,"diskwrite":14131200,"name":"alpine-docker","maxdisk":2040373248,"template":0,"uptime":727114,"node":"server5","vmid":109,"tags":"alpine;community-script;docker","id":"lxc/109","cpu":0.00124031986717544,"netout":93055,"maxmem":1073741824,"mem":69939200,"status":"running","diskread":855281664,"disk":960909312,"type":"lxc"},{"name":"OKD-Admin-Client-Machine","diskwrite":6828128768,"maxdisk":25232932864,"template":0,"vmid":110,"node":"server5","uptime":607539,"memhost":4006914048,"maxcpu":2,"netin":2781933394,"cpu":0.00271224248820189,"mem":4006914048,"maxmem":8589934592,"status":"running","netout":1848017849,"diskread":2401921428,"disk":0,"type":"qemu","id":"qemu/110"},{"tags":"boyang","id":"lxc/111","cpu":0.000451025406245615,"mem":200962048,"status":"running","maxmem":536870912,"netout":60008211,"diskread":893276160,"disk":1253220352,"type":"lxc","memhost":0,"maxcpu":2,"netin":343391832,"name":"PowerDNS","diskwrite":241131520,"maxdisk":8350298112,"template":0,"vmid":111,"node":"server5","uptime":727114},{"netout":1154995792,"maxmem":34359738368,"mem":34390517760,"status":"running","cpu":0.0752986320787049,"type":"qemu","diskread":5603250150,"disk":0,"tags":"boyang","id":"qemu/112","maxdisk":161061273600,"diskwrite":445671669760,"name":"okd4sno-vm","uptime":590388,"node":"server5","vmid":112,"template":0,"memhost":34390517760,"netin":31663527486,"maxcpu":32},{"memhost":0,"netin":0,"maxcpu":4,"maxdisk":85899345920,"diskwrite":0,"name":"vllm","uptime":0,"node":"server4","vmid":113,"template":0,"tags":"boyang","id":"qemu/113","netout":0,"mem":0,"maxmem":12884901888,"status":"stopped","cpu":0,"type":"qemu","disk":0,"diskread":0},{"memhost":2195421184,"netin":1010381686,"maxcpu":2,"maxdisk":8589934592,"diskwrite":2377115648,"name":"vyos","uptime":727183,"node":"server5","vmid":114,"template":0,"tags":"boyang","id":"qemu/114","netout":36193927038,"maxmem":2147483648,"mem":882741248,"status":"running","cpu":0.0141036609386498,"type":"qemu","disk":0,"diskread":449514496},{"template":0,"uptime":727114,"node":"server5","vmid":115,"diskwrite":23066398720,"name":"netbox","maxdisk":10464022528,"maxcpu":2,"netin":307095424,"memhost":0,"diskread":1009512448,"disk":2518196224,"type":"lxc","cpu":0.00248063973435088,"netout":37246,"mem":1126170624,"status":"running","maxmem":4294967296,"id":"lxc/115","tags":"boyang"},{"maxdisk":4143677440,"name":"netbox","diskwrite":6047084544,"vmid":116,"uptime":546712,"node":"server1","template":0,"memhost":0,"netin":544031459,"maxcpu":2,"maxmem":2147483648,"mem":1422086144,"status":"running","netout":5267192,"cpu":0.00221263081122382,"type":"lxc","disk":2141569024,"diskread":929480704,"tags":"community-script;network","id":"lxc/116"},{"id":"lxc/118","diskread":4595736576,"disk":1524928512,"type":"lxc","cpu":0,"maxmem":536870912,"mem":71110656,"status":"running","netout":851110,"maxcpu":1,"netin":452770989,"memhost":0,"template":0,"vmid":118,"uptime":443281,"node":"server3","name":"test","diskwrite":5251842048,"maxdisk":8350298112},{"id":"qemu/122","type":"qemu","diskread":0,"disk":0,"netout":0,"status":"stopped","mem":0,"maxmem":2147483648,"cpu":0,"netin":0,"maxcpu":2,"memhost":0,"uptime":0,"node":"server5","vmid":122,"template":1,"maxdisk":25232932864,"diskwrite":0,"name":"ubuntu-2404-cloud"}]}
    ```

<!--list-separator-->

-  List zones

    ```console
    yanboyang713@yanboyang713-Standard-PC-i440FX-PIIX-1996:~$ curl -k -sS -H 'Authorization: PVEAPIToken=netsync@pve!netbox-sync=7679bb8a-6b14-4fe1-aa8a-6d0f6a093b32' 'https://192.168.1.11:8006/api2/json/cluster/sdn/zones'
    {"data":[{"controller":"evpn","advertise-subnets":1,"dnszone":"pve.internal.lab","digest":"4b23b25710bbc529f739209ab68842780480ba6e","exitnodes-primary":"server5","mtu":1450,"dns":"powerdns1","exitnodes":"server2,server5","mac":"BC:24:11:B6:FE:2C","ipam":"netbox","type":"evpn","vrf-vxlan":10000,"reversedns":"powerdns1","zone":"evpntest"}]}
    ```

<!--list-separator-->

-  List vnets

    ```console
    yanboyang713@yanboyang713-Standard-PC-i440FX-PIIX-1996:~$ curl -k -sS -H 'Authorization: PVEAPIToken=netsync@pve!netbox-sync=7679bb8a-6b14-4fe1-aa8a-6d0f6a093b32' 'https://192.168.1.11:8006/api2/json/cluster/sdn/vnets'
    {"data":[{"zone":"evpntest","type":"vnet","digest":"0d50206d52321fc74f14905f84c651587d92f877","tag":100,"vnet":"testnet1"}]}
    ```


### Create a [Proxmox Linux Containers (LXC)]({{< relref "2023-12-03-143617-proxmox_lxc.md" >}}) for netbox-pve-sync {#create-a-proxmox-linux-containers--lxc----2023-12-03-143617-proxmox-lxc-dot-md--for-netbox-pve-sync}


#### Create an LXC **runner** (Proxmox UI) {#create-an-lxc-runner--proxmox-ui}

-   [PVE Container Images Download]({{< relref "2023-12-03-143617-proxmox_lxc.md#pve-container-images-download" >}}) → download debian-12-standard_\*
-   Click [create a CT Container]({{< relref "2023-12-03-143617-proxmox_lxc.md#steps-for-create-a-new-lxc-container" >}})
    -   General: Hostname pve-netbox-sync
    -   Template: pick the Debian 12 template
    -   Disks: 8–16 GB
    -   CPU/Memory: 1–2 cores, 512 MB–1 GB RAM
    -   Network: bridge vmbr0, static IP
        -   IP: 192.168.1.27/24
        -   Hostname: pve-netbox-sync
        -   Gateway: 192.168.1.4
    -   Unprivileged: enabled (recommended)
    -   Start after created: enabled


#### Inside the LXC: install netbox-pve-sync {#inside-the-lxc-install-netbox-pve-sync}

```bash
apt update
apt upgrade

apt install -y python3 python3-venv python3-pip ca-certificates git

git clone https://github.com/yanboyang713/netbox-pve-sync.git /opt/netbox-pve-sync
python3 -m venv /opt/netbox-pve-sync/.venv
/opt/netbox-pve-sync/.venv/bin/pip install --upgrade pip
/opt/netbox-pve-sync/.venv/bin/pip install -e /opt/netbox-pve-sync
```


#### Configure environment variables (use an env file) {#configure-environment-variables--use-an-env-file}

-   [Get NetBox API Token](#create-netbox--2025-11-30-174834-netbox-dot-md--api-token)
-   [Get Cluster ID when created the NetBox Cluster](#create-the-netbox--2025-11-30-174834-netbox-dot-md--cluster-that-will-hold-the-vms)
-   [Get a dedicated user for Proxmox VE (PVE) and NetBox sync](#create-a-dedicated-user-for-proxmox-ve--pve----20230228043925-proxmox-ve-dot-md--and-netbox--2025-11-30-174834-netbox-dot-md--sync)
-   [Get API Tokens for Proxmox VE (PVE) and NetBox sync](#create-api-tokens-for-proxmox-ve--pve----20230228043925-proxmox-ve-dot-md--and-netbox--2025-11-30-174834-netbox-dot-md--sync)

Create /etc/netbox-pve-sync.env in the LXC:

```file
NB_API_URL=https://netbox.testbed.com
NB_API_TOKEN=...             # NetBox API token
NB_CLUSTER_ID=...            # numeric ID from cluster URL

PVE_API_HOST=192.168.1.11
PVE_API_USER=netsync@pve
PVE_API_TOKEN=netbox-sync
PVE_API_SECRET=...           # Proxmox API token secret

REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
```


#### Fix NetBox HTTPS (self-signed cert) {#fix-netbox-https--self-signed-cert}

-   Pre-require: [NetBox Re-issue a self-signed cert that works for IP and/or hostname]({{< relref "2025-12-14-141844-netbox_installation.md#id-3fb5adf6-b8c8-43cd-9726-c1d126458e4e-netbox-re-issue-a-self-signed-cert-that-works-for-ip-and-or-hostname" >}})

Recommended: trust your NetBox cert in the LXC (so Python requests works without disabling TLS):

-   Copy your NetBox cert/CA to the LXC as /usr/local/share/ca-certificates/netbox.crt
    ```bash
      mkdir -p /usr/local/share/ca-certificates
      # single command
      openssl s_client -connect 192.168.1.26:443 -servername netbox.testbed.com -showcerts </dev/null 2>/dev/null \
    | awk 'n==0 && /BEGIN CERTIFICATE/{n=1} n{print} /END CERTIFICATE/{exit}' \
    > /usr/local/share/ca-certificates/netbox.crt
    ```
-   Quick sanity check
    ```bash
    root@pve-netbox-sync:~# openssl x509 -in /usr/local/share/ca-certificates/netbox.crt -noout -subject -issuer
    subject=C = US, O = NetBox, OU = Certificate, CN = netbox.testbed.com
    issuer=C = US, O = NetBox, OU = Certificate, CN = netbox.testbed.com
    ```
-   Then run:

<!--listend-->

```bash
update-ca-certificates
```

Alternative quick workaround: set **REQUESTS_CA_BUNDLE=/path/to/netbox-ca.pem** in the env file, but system CA is cleaner.

-   Edit /etc/hosts
    ```file
    192.168.1.26 netbox.testbed.com
    ```
-   Verify

<!--listend-->

```bash
getent hosts netbox.testbed.com
```


#### Run a manual test {#run-a-manual-test}

```bash
set -a; . /etc/netbox-pve-sync.env; set +a
/opt/netbox-pve-sync/.venv/bin/nbpxsync

/opt/netbox-pve-sync/.venv/bin/nbpxsync --help
```

If it fails, the two most common blockers are:

-   Proxmox token can’t list VMs: curl -k -sS -H 'Authorization: PVEAPIToken=netsync@pve!netbox-sync=SECRET' 'https://192.168.1.11:8006/api2/json/cluster/resources?type=vm' must return data.
-   NetBox prerequisites missing: cluster exists, Proxmox nodes exist as Devices with matching names, and required custom fields exist.


## Reference List {#reference-list}

1.  <https://github.com/netdevopsbr/netbox-proxbox>
2.  <https://github.com/creekorful/netbox-pve-sync>
3.  <https://netboxlabs.com/blog/automate-proxmox-virtual-machine-configuration-netbox-flask-application/>
4.  <https://github.com/netboxlabs/netbox-proxmox-automation/>
5.  <https://netboxlabs.com/blog/automate-proxmox-virtual-machine-configuration-netbox-ansible-awx-tower-automation-platform-aap/>
