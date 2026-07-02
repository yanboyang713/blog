---
title: "Setting up Static IP address on Ubuntu Server"
draft: false
---

## Pre-required {#pre-required}

-   [IP addresses]({{< relref "20230529165243-ip_addresses.md" >}})
-   [ubuntu]({{< relref "20230220222545-ubuntu.md" >}})


## Alternative method {#alternative-method}

-   [Set a static IP address based on nmcli]({{< relref "2024-02-05-143756-nmcli.md#set-a-static-ip-address" >}})


## Setting up Static IP address on Ubuntu Server {#setting-up-static-ip-address-on-ubuntu-server}

Login to your Ubuntu server 22.04, look for the netplan configuration file. It is located under /etc/netplan directory.

```console
boyang@k8s-master:~$ cd /etc/netplan/
boyang@k8s-master:/etc/netplan$ ls
00-installer-config.yaml
```

Run below cat command to view the contents of ‘00-installer-config.yaml’

**Note**: Name of configuration file may differ as your per setup. As it is an yaml file, so make sure to maintain the indentation and syntax while editing.

```console
boyang@k8s-master:/etc/netplan$ cat 00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens18:
      dhcp4: true
  version: 2
```

As per above output, it says that we have ens18 interface and it is getting ip from dhcp server. Alternate way to view interface name is via ip command.

Now, to configure static ip in place of dhcp, edit netplan configuration file using vi or nano editor and add the following content.

```console
$ sudo vi 00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
    renderer: networkd
    ethernets:
      ens18:
        addresses:
          - 192.168.88.216/24
        nameservers:
          addresses: [192.168.88.1, 8.8.8.8]
        routes:
          - to: default
            via: 192.168.88.1
    version: 2
```

save and close (:wq) the file.

In the above file we have used following,

-   ens18 is the interface name
-   addresses are used to set the static ip
-   nameservers used to specify the DNS server ips
-   routes used to specify the default gateway

**Note**: Change the IP details and interface name as per your environment.

To make above changes into the effect the apply these changes using following netplan command,

```bash
sudo netplan apply
```

Run following ip command to view the ip address on interface,

```console
boyang@k8s-master:~$ ip addr show ens18
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 76:0c:f0:43:29:46 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    inet 192.168.88.216/24 brd 192.168.88.255 scope global ens18
       valid_lft forever preferred_lft forever
    inet6 fe80::740c:f0ff:fe43:2946/64 scope link
       valid_lft forever preferred_lft forever
```

```console
boyang@k8s-master:~$ ip route show
default via 192.168.88.1 dev ens18 proto static
192.168.88.0/24 dev ens18 proto kernel scope link src 192.168.88.216
```

```console
boyang@k8s-master:~$ ping -c 3 google.com
PING google.com (74.125.138.100) 56(84) bytes of data.
64 bytes from yi-in-f100.1e100.net (74.125.138.100): icmp_seq=1 ttl=105 time=54.6 ms
64 bytes from yi-in-f100.1e100.net (74.125.138.100): icmp_seq=2 ttl=105 time=104 ms
64 bytes from yi-in-f100.1e100.net (74.125.138.100): icmp_seq=3 ttl=105 time=15.7 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 15.705/57.951/103.584/35.956 ms
```

Perfect, above commands’ output confirms that static ip and route has been configured successfully.


## Reference List {#reference-list}

1.  <https://www.linuxtechi.com/static-ip-address-on-ubuntu-server/>
