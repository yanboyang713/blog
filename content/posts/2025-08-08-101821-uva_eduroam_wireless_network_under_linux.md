---
title: "UVA Wireless Network Connection"
date: 2025-08-08
draft: false
---

[University of Virginia]({{< relref "2025-04-11-013919-university_of_virginia.md" >}}) have two wifi networks, eduroam and wahoo.


## Eduroam {#eduroam}


### Get a UVA's Personal Certificate (.p12) {#get-a-uva-s-personal-certificate--dot-p12}

Linux or if you need the actual .p12 file: Use the SecureW2 “Limited” portal to generate and download your UVA Digital Certificate (.p12). You’ll set a passphrase during creation, then import it into NetworkManager/Firefox.

-   Reference:

<https://virginia.service-now.com/its?id=itsweb_kb_article&sys_id=58aafbcfdbf6c744f032f1f51d961927>

-   Steps

Point a web browser at <https://cloud.securew2.com/public/82116/limited/?device=Unknown> and follow the instructions there.
![](https://galileo.phys.virginia.edu/compfac/faq/new-cert-gen-1.png)

**Note:** that you’ll be asked to choose a passphrase for the certificate. The passphrase must be shorter than 15 characters. If you attempt to enter a longer passphrase, you’ll see a "passphrase invalid" message.
![](https://galileo.phys.virginia.edu/compfac/faq/new-cert-gen-2.png)
At the end, this process should download a file with a name ending in ".p12".


### Get get the USHER Certificate: {#get-get-the-usher-certificate}

Point a web browser at <https://discovery.phys.virginia.edu/compfac/faq/ca.pem> and download a file named "ca.pem".

```bash
wget https://discovery.phys.virginia.edu/compfac/faq/ca.pem -O ca.pem
```


### Configure Your Eduroam Connection {#configure-your-eduroam-connection}

Click the network icon (nm-applet) on your task bar. Configure your Eduroam connection as shown here.

```file

*Warning* You might need to put your p12 file into both the "User Certificate" and "Private Key" fields to make the other fields available, or to make it possible to save the configuration.

Note other caveats below the figure:
[[https://galileo.phys.virginia.edu/compfac/faq/eduroam-wifi-config-dialog.png]]

Your "identity" should be in the form mst3k@virginia.edu. Note that all letters must be in lower case.

The "password" should be the passphrase you entered when you created your certificate, above.

Save your changes.
```


## wahoo {#wahoo}

Only iOS, Android, macOS and Windows devices can run the Network Setup Tool and connect to the eduroam wireless network. Other devices, such as game console, streaming video players, Linux computers, and voice assistants must connect to the wahoo wireless network. UVA Information Technology Services (ITS) does not support these devices and provides instructions only as a courtesy.

Instructions for connecting to wahoo are at [https://in.virginia.edu/wahoo](https://in.virginia.edu/wahoo)
**NOTE**:

-   Be aware that wahoo is a hidden wireless network whose network name does not display in available wireless network lists and must be typed in.
-   Devices will need to be completely powered down and restarted after registration to connect to wahoo.
-   You may want to register your device from a computer. Note that you will need the console or streaming device's MAC (Hardware) Address to do this and set the wireless flag to "Yes" on the Network Registration page at <https://netreg.itc.virginia.edu> (requires a UVA VPN - More Secure or UVAnywhere or More Secure).
-   Be sure to select "Yes" for "This is a wireless device" on the Network Registration page.
-   You can check/change this setting after registering your device by logging into NetReg, selecting "View and edit existing device registrations", selecting your device by clicking on the MAC address link, and looking at the "Wireless device" setting.

<!--listend-->

```bash
nmcli c up "wahoo"
```


### Enable autoconnect retries {#enable-autoconnect-retries}

This tells NetworkManager to try again quickly after a disconnect:

```bash
nmcli connection modify "wahoo" connection.autoconnect yes
nmcli connection modify "wahoo" connection.autoconnect-retries 0
```

0 here means “retry forever.” If you want a limit, use a positive number.


### Increase its priority {#increase-its-priority}

```bash
nmcli connection modify "wahoo" connection.autoconnect-priority 10
```


### Lower ping timeout so NM detects drops faster {#lower-ping-timeout-so-nm-detects-drops-faster}

NetworkManager setting that makes it detect a network drop sooner, so it can reconnect faster.
Specifically:

-   The property is connection.ip-ping-timeout.
-   When set, NetworkManager will send ICMP pings to the gateway (or configured IP) to verify the connection.
-   If the pings fail for the duration you set (in seconds), NM will mark the connection as down and trigger an immediate reconnect attempt.

A) Use your gateway (recommended):

```bash
nmcli connection modify "wahoo" ipv4.may-fail no
```

```bash
GW="$(nmcli -g IP4.GATEWAY connection show "wahoo")"
nmcli connection modify "wahoo" connection.ip-ping-addresses "$GW" connection.ip-ping-timeout 10
```

B) Use public IP(s) (only if ICMP isn’t blocked):

```bash
nmcli connection modify "wahoo" \
  connection.ip-ping-addresses 1.1.1.1,8.8.8.8 \
  connection.ip-ping-timeout 10
```

Then reactivate the connection so it takes effect:

```bash
nmcli connection down "wahoo" && nmcli connection up "wahoo"
```

means:
If "wahoo" loses connectivity and the pings fail for 10 seconds,
NetworkManager won’t wait for higher-level timeouts — it will declare the connection dead and reconnect right away.
By default, this is usually disabled (0), meaning NM waits for the interface’s own link detection or DHCP failure, which can be slow (sometimes 30–60+ seconds).


#### When to use it {#when-to-use-it}

-   On Wi-Fi connections: useful if the AP drops but the interface still thinks it’s “connected.”
-   On VPN connections: helps detect when the tunnel silently breaks.


#### Caveats {#caveats}

-   It adds a bit of extra ping traffic.
-   If your network blocks ICMP, NM might think it’s always down — so only use it if pings are allowed.


### Make it persistent across sleep/wake cycles {#make-it-persistent-across-sleep-wake-cycles}

If you suspend your machine, NetworkManager will try to reconnect on wake — but if you want to be aggressive:

```bash
nmcli connection modify "wahoo" connection.lldp disable
```

This avoids LLDP detection delays that sometimes slow Wi-Fi reconnects.


## Reference List {#reference-list}

1.  <https://galileo.phys.virginia.edu/compfac/faq/linux-eduroam.html>
