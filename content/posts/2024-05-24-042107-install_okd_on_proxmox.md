---
title: "Deploying OKD Single Node on Proxmox"
draft: false
---

This guide documents the complete setup of [OKD]({{< relref "20230518171737-okd.md" >}}) (The Community Distribution of [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) that powers Red Hat OpenShift) as a Single Node Operator (SNO) cluster on a virtual machine hosted by [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}).

[OKD Bootstrap]({{< relref "2024-05-24-051502-okd_bootstrap.md" >}})


## Prerequisites {#prerequisites}

-   [Proxmox Testbed]({{< relref "2025-04-10-010004-data_center_testbed_design.md" >}}) with [CPU virtualization (Intel VT-x / AMD-V)]({{< relref "2025-12-19-212625-hardware_virtualization.md#cpu-virtualization--intel-vt-x-amd-v" >}}) enabled in the BIOS/UEFI.
-   Sufficient Resources:
    -   CPU: Minimum 8 vCPUs recommended for SNO.
    -   RAM: Minimum 32GB RAM recommended for SNO.
    -   Storage: Minimum 120GB-150GB fast storage (SSD/NVMe) for the OKD VM.
-   Administrative Machine (Client): A Linux machine (e.g., Ubuntu, Fedora) to run openshift-install, oc, podman, and other client tools. This will be referred to as your “Admin Client Machine”.
-   Red Hat Pull Secret: Obtain a pull secret from [Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret) (a free Red Hat developer account is sufficient). This is needed for some certified operators and images.
-   Admin access to the local [PowerDNS]({{< relref "2024-05-25-025431-powerdns.md" >}}) with reverse DNS
-   An [ssh public key]({{< relref "20230901181155-ssh_key.md" >}}).
-   [Kea DHCP]({{< relref "2025-12-11-210940-kea_dhcp.md" >}}) must works, because every nodes must set hostnames, could be via DHCP reservations.


## Phase 1: Preparation on Admin Client Machine {#phase-1-preparation-on-admin-client-machine}

All commands in this phase are executed on your Admin Client Machine - [cloud init]({{< relref "2025-12-12-081925-cloud_init.md" >}}) enabled VM, not [Linux Containers (LXC)]({{< relref "2023-12-03-143424-lxc.md" >}}) (Disk: 20G; RAM: 8G(8192M); CPU: 2 cores).

-   OS: Ubuntu 24.04
-   CPU Type: x86-64-v3
-   IP: 192.168.1.50/24
-   Gateway: 192.168.1.4
-   PowerDNS:
    -   DNS Domain: okd.admin.testbed.com
    -   DNS Servers: 192.168.1.23


### Set Environment Variables {#set-environment-variables}

Define the OKD version and architecture for consistency.

```bash
export OKD_VERSION=4.20.0-okd-scos.15
export ARCH="$(uname -m)"   # x86_64 on Intel/AMD
echo "$ARCH"
```

**NOTE**:

1.  Check for the latest stable SCOS release: <https://github.com/okd-project/okd/releases>
2.  **ec** mean preview version


### Download OpenShift Client (oc) {#download-openshift-client--oc}

```bash
curl -L "https://github.com/okd-project/okd/releases/download/${OKD_VERSION}/openshift-client-linux-${OKD_VERSION}.tar.gz" -o oc.tar.gz
tar zxf oc.tar.gz
chmod +x oc kubectl # kubectl is also included
sudo mv oc kubectl /usr/local/bin/ # Optional: Move to PATH for global access
oc version
```


### Download OpenShift Installer (openshift-install) {#download-openshift-installer--openshift-install}

```bash
curl -L "https://github.com/okd-project/okd/releases/download/${OKD_VERSION}/openshift-install-linux-${OKD_VERSION}.tar.gz" -o openshift-install-linux.tar.gz
tar zxvf openshift-install-linux.tar.gz
chmod +x openshift-install
sudo mv openshift-install /usr/local/bin/ # Optional: Move to PATH
openshift-install version
```


### Create a directory for your installation files and the install-config.yaml file. {#create-a-directory-for-your-installation-files-and-the-install-config-dot-yaml-file-dot}

```bash
mkdir okd-sno-install
cd okd-sno-install
```


#### Get CentOS Stream CoreOS / SCOS Live ISO {#get-centos-stream-coreos-scos-live-iso}

The installer will determine the correct FCOS ISO URL matching the OKD version.

```bash
export ISO_URL="$(openshift-install coreos print-stream-json | jq -r '.architectures.x86_64.artifacts.metal.formats.iso.disk.location')"
export ISO_SHA256="$(openshift-install coreos print-stream-json | jq -r '.architectures.x86_64.artifacts.metal.formats.iso.disk.sha256')"
echo "Downloading ISO from: ${ISO_URL}"
echo "Downloading ${ISO_SHA256}"
```

```bash
curl -L "${ISO_URL}" -o fcos-live.iso
# compute the ISO hash
sha256sum fcos-live.iso
# verify against the expected hash (prints "OK" if it matches)
echo "${ISO_SHA256}  fcos-live.iso" | sha256sum -c -
```

**Note**: This fcos-live.iso will be uploaded to Proxmox later.


#### Prepare install-config.yaml {#prepare-install-config-dot-yaml}

please, perpare [ssh public key]({{< relref "20230901181155-ssh_key.md" >}})
Create **install-config.yaml** with the following content:

```yaml
apiVersion: v1
baseDomain: okd.lan # Your local base domain
metadata:
  name: okd4sno    # Your cluster name
compute:
- name: worker
  replicas: 0     # Essential for SNO
controlPlane:
  name: master
  replicas: 1     # Essential for SNO
networking:
  networkType: OVNKubernetes
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 192.168.1.0/24 # Your Proxmox VM network subnet
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {} # For bare-metal/VM installations not managed by a cloud provider
bootstrapInPlace:
  # IMPORTANT: Identify your Proxmox VM's target disk for installation.
  # This can be /dev/vda, /dev/sda, or a more stable WWN path.
  # Example for a VirtIO disk, often /dev/vda:
  installationDisk: /dev/sda
  # Example using WWN (more robust, get this from Proxmox VM's disk details or from FCOS live env):
  # installationDisk: /dev/disk/by-id/wwn-0x64cd98f04fde100024684cf3034da5c2
pullSecret: '<PASTE_YOUR_PULL_SECRET_JSON_HERE>' # Replace with your actual pull secret
sshKey: |
  ssh-rsa AAAA...your_public_ssh_key_here # Replace with your public SSH key

```

**Important Notes** for install-config.yaml:

-   **baseDomain** and **metadata.name**: These will form your cluster’s FQDNs (e.g., **api.okd4sno.okd.lan**).
-   **machineNetwork.cidr**: Ensure this matches the subnet your OKD VM will reside in.
-   installationDisk:
    -   For Proxmox VirtIO disks, this is typically /dev/vda. For SCSI, it might be /dev/sda.
    -   Using _dev/disk/by-id/wwn-0x... is more robust if disk names might change. You can identify the correct WWN or device path by booting the FCOS Live ISO on the target VM and using commands like lsblk or ls /dev/disk/by-id_.
    -   pullSecret: Paste the entire JSON string from Red Hat.
    -   sshKey: Your public SSH key to access the core user on the FCOS node.


### Generate Single Node [Ignition]({{< relref "2025-12-30-021315-ignition.md" >}}) Configuration {#generate-single-node-ignition--2025-12-30-021315-ignition-dot-md--configuration}

This command uses the install-config.yaml in the current directory.

```bash
# Still in okd-sno-install directory
openshift-install create single-node-ignition-config
```

This will create **bootstrap-in-place-for-live-iso.ign** in the current directory.


### Embed Ignition into FCOS Live ISO {#embed-ignition-into-fcos-live-iso}

We need **coreos-installer** for this, which can be run via [Podman]({{< relref "2025-10-23-205812-podman.md" >}}).

```bash
# Ensure you are in the directory containing fcos-live.iso and the bootstrap ignition file
# (which should be okd-sno-install if you followed above)

podman run --privileged --pull always --rm \
  -v /dev:/dev -v /run/udev:/run/udev -v "$PWD":"$PWD" -w "$PWD" \
  quay.io/coreos/coreos-installer:release \
  iso ignition embed -fi bootstrap-in-place-for-live-iso.ign fcos-live.iso

```

"Writing manifest to image destination" means success.

**NOTE**:

1.  If you don't want to embed Ignition into ISO, you also could manual install, all nodes use same ISO image.

<!--listend-->

```bash
sudo coreos-installer install /dev/sda --ignition-file /path/to/master.ign --copy-network
sudo reboot
```

1.  --copy-network tells coreos-installer to carry over the network configuration that is active in the Live environment (the ISO booted system) into the installed OS on disk.


### Embed static networking (option, if does not have DHCP) {#embed-static-networking--option-if-does-not-have-dhcp}

Create **static.nmconnection** (adjust interface-name, IP, gateway, DNS):

```file
[connection]
id=static
type=ethernet
interface-name=ens18
autoconnect=true

[ipv4]
method=manual
address1=192.168.1.51/24,192.168.1.4
dns=192.168.1.23;8.8.8.8;
dns-search=okd.lan;
may-fail=false

[ipv6]
method=disabled
```

How to confirm the NIC name (ens18 vs something else) when booted into the live ISO:

```bash
ip -br link
```

Embed static networking

```bash
podman run --privileged --pull always --rm \
  -v /dev:/dev -v /run/udev:/run/udev -v "$PWD":"$PWD" -w "$PWD" \
  quay.io/coreos/coreos-installer:release \
  iso network embed -f -k static.nmconnection fcos-live.iso
```

"Writing manifest to image destination" means success.


## Phase 2: Proxmox VM Creation and FCOS Installation {#phase-2-proxmox-vm-creation-and-fcos-installation}


### Upload Modified ISO to Proxmox {#upload-modified-iso-to-proxmox}

Upload the modified fcos-live.iso (the one with the embedded Ignition config) to your Proxmox VE “ISO Images” storage (e.g., local storage).

```bash
scp -p ./fcos-live.iso root@192.168.1.15:/var/lib/vz/template/iso/
```


### Create the OKD Virtual Machine in Proxmox {#create-the-okd-virtual-machine-in-proxmox}

-   General:
    -   Name: okd4sno-vm (or similar)
    -   Guest OS Type: Linux, Version: 6.x - 2.6 Kernel (or latest)
-   System:
    -   Machine: q35
    -   BIOS: OVMF (UEFI)
    -   EFI Storage: Select your Proxmox storage for the EFI disk, and Uncheck Pre-Enroll keys.
    -   Enable Qemu Agent.
-   Disks:
    -   Create a virtual hard disk (VirtIO Block or SCSI) with at least 120GB-150GB on fast storage. This is the disk you specified in installationDisk (e.g., /dev/sda).
-   CPU:
    -   Cores: 8 (or more)
    -   Type: host (for best performance)
-   Memory:
    -   32768 MiB (32GB) or more. Disable “Ballooning Device”.
-   Network:
    -   Model: VirtIO (paravirtualized)
    -   Bridge: Your Proxmox bridge connected to your LAN (e.g., vmbr0).
    -   IP: 192.168.1.51/24
    -   Gateway: 192.168.1.4
    -   [PowerDNS]({{< relref "2024-05-25-025431-powerdns.md" >}}):
    -   DNS Domain: okd.lan
    -   DNS Servers: 192.168.1.23
-   CD/DVD Drive:
    -   Select the modified fcos-live.iso you uploaded.
-   Boot Order:
    -   Set the CD/DVD drive (with the FCOS ISO) as the first boot device.

**NOTE**: before you start VM, please set-up [DNS](#phase-3-dns-and-cluster-access) first.


## Phase 3: DNS and Cluster Access {#phase-3-dns-and-cluster-access}


### DNS Setup ([PowerDNS]({{< relref "2024-05-25-025431-powerdns.md" >}})) {#dns-setup--powerdns-2024-05-25-025431-powerdns-dot-md}

For this guide, let’s assume the OKD VM gets the IP 192.168.1.51 (via DHCP reservation or manually configured static IP if you adapted the Ignition).


#### Create the okd.lan zone and add the OKD records {#create-the-okd-dot-lan-zone-and-add-the-okd-records}

```console
root@PowerDNS:~# pdnsutil zone create okd.lan ns1.okd.lan
Dec 30 09:46:59 [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
Creating empty zone 'okd.lan'
Also adding one NS record
```


#### Add the required records (SNO points them all at the single node IP) {#add-the-required-records--sno-points-them-all-at-the-single-node-ip}

```console
root@PowerDNS:~# pdnsutil rrset add okd.lan api.okd4sno.okd.lan A 192.168.1.51
Dec 30 09:49:13 [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
New rrset:
api.okd4sno.okd.lan. 3600 IN A 192.168.1.51
root@PowerDNS:~# pdnsutil rrset add okd.lan api-int.okd4sno.okd.lan A 192.168.1.51
Dec 30 09:49:48 [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
New rrset:
api-int.okd4sno.okd.lan. 3600 IN A 192.168.1.51
root@PowerDNS:~# pdnsutil rrset add okd.lan '*.apps.okd4sno.okd.lan' A 192.168.1.51
Dec 30 09:50:22 [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
New rrset:
*.apps.okd4sno.okd.lan. 3600 IN A 192.168.1.51
```


#### Add [PowerDNS Recursor]({{< relref "2024-05-25-025431-powerdns.md#configure-powerdns-recursor" >}}) Zone correct {#add-powerdns-recursor--2024-05-25-025431-powerdns-dot-md--zone-correct}


#### (Optional but nice) Add a node hostname too {#optional-but-nice--add-a-node-hostname-too}

```console
root@PowerDNS:~# pdnsutil rrset add okd.lan okd4sno.okd.lan A 192.168.1.51
Dec 30 09:52:38 [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
New rrset:
okd4sno.okd.lan. 3600 IN A 192.168.1.51
```

OKD 4.4+ does not require separate etcd SRV records in DNS, which simplifies your lab setup.


#### Restart {#restart}

```bash
systemctl restart pdns
systemctl status pdns --no-pager

systemctl restart pdns-recursor
systemctl status pdns-recursor --no-pager
```


#### Validate before installing {#validate-before-installing}

```bash
dig +short api.okd4sno.okd.lan @192.168.1.23
dig +short api-int.okd4sno.okd.lan @192.168.1.23
dig +short console-openshift-console.apps.okd4sno.okd.lan @192.168.1.23
```


### Set hostname {#set-hostname}

```bash
hostnamectl
```

```bash
sudo hostnamectl set-hostname okd4sno.okd.lan
```


### Monitor Installation {#monitor-installation}


#### Monitor Installation from SNO host {#monitor-installation-from-sno-host}

-   Start the VM. It will boot from the modified FCOS Live ISO.
    On CoreOS, you typically cannot log in from the VM console with a password. The system is designed for SSH key–based login.
    Login method: SSH as core from your Admin Client
    ```console
    ubuntu@OKD-Admin-Client-Machine:~/okd-sno-install$ ssh -i ~/.ssh/id_rsa core@192.168.1.51
    The authenticity of host '192.168.1.51 (192.168.1.51)' can't be established.
    ED25519 key fingerprint is SHA256:+FaXbzR18oR+HTOUlGDNAeBblhjc8EYkRb4pEQo8ys8.
    This key is not known by any other names.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '192.168.1.51' (ED25519) to the list of known hosts.
    This is the bootstrap node; it will be destroyed when the master is fully up.

    The primary services are release-image.service followed by bootkube.service. To watch their status, run e.g.

      journalctl -b -f -u release-image.service -u bootkube.service

    ```

Common patterns:

-   node-image-pull.service: downloads payload images (can take a while)
-   release-image.service: prepares the release image
-   bootkube.service: brings up the initial control-plane components

You are successfully logged in. That message is normal for bootstrap-in-place: the system starts in a temporary “bootstrap” mode and then transitions to the final single-node control-plane. Because the Ignition config is embedded and bootstrapInPlace.installationDisk is set, FCOS should automatically install itself to the specified disk (/dev/sda) and then reboot.


#### Watch the bootstrap-in-place services {#watch-the-bootstrap-in-place-services}

```bash
journalctl -b -f -u release-image.service -u bootkube.service -u node-image-pull.service
```

-

<!--listend-->

```console
Bootstrap completed, server is going to reboot.
The system will reboot at Thu 2025-12-25 05:12:26 UTC!
```


#### Monitor Installation from Admin Client Machine {#monitor-installation-from-admin-client-machine}

Once the OKD VM has booted from its hard disk and the FCOS installation + Ignition processing is complete, the OKD bootstrap process will start.

```bash
# On your Admin Client Machine, in the okd-sno-install directory
openshift-install wait-for bootstrap-complete --log-level=info
# This can take 20-40 minutes.
# Once bootstrap is complete:
openshift-install wait-for install-complete --log-level=info
# This can take another 30-60+ minutes.
```


### Web Console Access {#web-console-access}

After install-complete finishes:

-   Navigate to: <https://console-openshift-console.apps.okd4sno.okd.lan>
-   Login using:
    -   Username: kubeadmin
    -   Password: Found in okd-sno-install/auth/kubeadmin-password on your Admin Client Machine.


## DNS {#dns}


### Forward DNS definitions {#forward-dns-definitions}

```file
; OKD
haproxy                 IN      A       192.168.88.8
helper                  IN      A       192.168.88.8
helper.okd              IN      A       192.168.88.8
api.okd                 IN      A       192.168.88.8
api-int.okd             IN      A       192.168.88.8
*.apps.okd              IN      A       192.168.88.8
bootstrap.okd           IN      A       192.168.88.12
master0.okd             IN      A       192.168.88.9
master1.okd             IN      A       192.168.88.10
master2.okd             IN      A       192.168.88.11
```


### Reverse DNS {#reverse-dns}

```file
; okd
8   IN      PTR     haproxy.yanboyang.com.
8   IN      PTR     helper.yanboyang.com.
8   IN      PTR     helper.okd.yanboyang.com.
8   IN      PTR     api.okd.yanboyang.com.
8   IN      PTR     api-int.okd.yanboyang.com.
12   IN      PTR     bootstrap.okd.yanboyang.com.
9   IN      PTR     master0.okd.yanboyang.com.
10   IN      PTR     master1.okd.yanboyang.com.
11   IN      PTR     master2.okd.yanboyang.com.
```


## Reference List {#reference-list}

1.  <https://github.com/gardart/okd-proxmox-scripts>
2.  <https://github.com/pvelati/okd-proxmox-scripts>
3.  <https://github.com/pvelati/ansible-okd-proxmox>
4.  <https://www.pivert.org/deploy-openshift-okd-on-proxmox-ve-or-bare-metal-tutorial/>
5.  <https://andrearaponi.it/devops/deploy-okd-on-proxmox/>
6.  <https://docs.okd.io/latest/installing/installing_platform_agnostic/installing-platform-agnostic.html>
