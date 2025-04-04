---
title: "Linux Directory Structure"
draft: false
---

## Directories {#directories}


### Configuration files {#configuration-files}

Configuration files should be placed in the _etc directory. If there is more than one configuration file, it is customary to use a subdirectory in order to keep the /etc area as clean as possible. Use /etc/pkg where pkg is the name of the package (or a suitable alternative, eg, apache uses /etc/httpd_).


### General directory {#general-directory}

Package files should follow these general directory guidelines:

| directory               | description                          |
|-------------------------|--------------------------------------|
| /etc                    | System-essential configuration files |
| /usr/bin                | Binaries                             |
| /usr/lib                | Libraries                            |
| /usr/include            | Header files                         |
| /usr/lib/pkg            | Modules, plugins, etc.               |
| /usr/share/doc/pkg      | Application documentation            |
| /usr/share/info         | GNU Info system files                |
| /usr/share/licenses/pkg | Application licenses                 |
| /usr/share/man          | Manpages                             |
| /usr/share/pkg          | Application data                     |
| /var/lib/pkg            | Persistent application storage       |
| /etc/pkg                | Configuration files for pkg          |
| /opt/pkg                | Large self-contained packages        |


### Packages should not contain any of the following directories: {#packages-should-not-contain-any-of-the-following-directories}

/bin
/sbin
/dev
/home
/srv
/media
/mnt
/proc
/root
/selinux
/sys
/tmp
/var/tmp
/run


## Reference List {#reference-list}

1.  <https://wiki.archlinux.org/title/Arch_package_guidelines#Directories>
