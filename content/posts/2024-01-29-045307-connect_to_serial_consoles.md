---
title: "Connect to Serial Consoles"
draft: false
---

## Setting Up the USB to Serial Converter {#setting-up-the-usb-to-serial-converter}

Almost any USB serial converter you can find on the market is automatically recognized by Linux. Plug any USB converter into your computer and use the dmesg command to find out the file name of the device.

```console
dmesg
[20285.028974] usbserial: USB Serial support registered for FTDI USB Serial Device
[20285.029048] ftdi_sio 1-2:1.0: FTDI USB Serial Device converter detected
[20285.029091] usb 1-2: Detected FT232R
[20285.033145] usb 1-2: FTDI USB Serial Device converter now attached to ttyUSB0
```

You can use the serial converter recognized by your system via the device file /dev/ttyUSB0. Depending on the distribution you use, the device file is automatically created under the /dev directory, showing major, minor, and device types. For USB serial converters, the naming scheme is usually ttyUSB0, ttyUSB1, and ttyUSBX for each simultaneous translation.


## Access Authorization on Serial Devices {#access-authorization-on-serial-devices}

```console
yanboyang713@Meta-Scientific-Linux ~ % ls -l /dev/ttyUSB0
crw-rw---- 1 root uucp 188, 0 Feb  1 10:51 /dev/ttyUSB0
```

On examining the above output, you can tell:

-   The letter c at the beginning of the line denotes that this is a character-based device
-   The file owner is the root user and the user has read and write privileges
-   The group owner of the file is the uucp group and users included in this group also have read and write permissions
-   The rest of the users do not have any read and write rights on the file

If you're currently logged in as a regular user, who isn't a member of the **uucp** group, you cannot read and write to the USB serial converter. To solve this problem, you must either make the current user a member of the dialout group or edit the udev rule files on your system.

You can utilize the first technique for convenience. To begin, use the id command to determine which groups your user belongs to:

```console
yanboyang713@Meta-Scientific-Linux ~ % id
uid=1000(yanboyang713) gid=1000(yanboyang713) groups=1000(yanboyang713),969(docker),998(wheel)
```

Add your user to the group using the adduser or usermod command:

```bash
# For Fedora
sudo usermod -aG dialout USERNAME

# For Debian
sudo adduser USERNAME dialout

# For Arch
sudo usermod -a -G uucp USERNAME
```


## How to Connect to Serial Consoles on Linux {#how-to-connect-to-serial-consoles-on-linux}

When you need access to a computer or network console, you can refer to serial console applications. Usually, you require access over SSH to do so. However, from a software and hardware point of view, in some cases, it is also possible to access the console using only serial ports.


### Using GTKTerm {#using-gtkterm}

You can install the application on your system using the following commands:

```bash
# On Fedora, CentOS, and RHEL
sudo dnf -y install gtkterm

# On Debian and Ubuntu
sudo apt-get install gtkterm

# On Arch Linux
paru -S gtkterm
```

When you launch the app, you'll have to set the device name and speed parameters via the Configuration &gt; Port menu as follows:
![](https://static1.makeuseofimages.com/wordpress/wp-content/uploads/2022/07/gtkterm-gui-connection-rate-speed.jpg?q=50&fit=crop&w=1500&dpr=1.5)


## Reference List {#reference-list}

1.  <https://www.makeuseof.com/connect-to-serial-consoles-on-linux/>
