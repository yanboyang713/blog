---
title: "arch linux package downgrade"
draft: false
---

## Install downgrade utility in [Arch Linux]({{< relref "20230220222636-arch_linux.md" >}}) {#install-downgrade-utility-in-arch-linux--20230220222636-arch-linux-dot-md}

This package is not available in the official repositories. You can either install it from AUR or from the unofficial archlinuxfr repository. First we will see how to install it from AUR.

Install downgrade from AUR
The downgrade package is available in AUR, so you can install it using any AUR helper programs such as Paru or Yay.

Using Paru:

$ paru -S downgrade
Using Yay:

$ yay -S downgrade
Install downgrade from archlinuxfr repository
A word of caution: The official Arch Linux Developers and the Trusted Users do not perform tests of any sort to verify the contents of unofficial repositories. You must decide whether to trust their maintainers and you take full responsibility for any consequences of using any unofficial repository.

To add archlinuxfr repository, edit /etc/pacman.conf file:

$ sudo nano /etc/pacman.conf
Add the following lines:

[archlinuxfr]
Server = <http://repo.archlinux.fr/$arch>
Save and close the file.

Update the repositories with command:

$ sudo pacman -Syu
Then install "Downgrade" utility using the following command from your Terminal:

$ sudo pacman -S downgrade
Downgrade a package in Arch Linux
The typical usage of "downgrade" command is:

$ sudo downgrade [PACKAGE, ...] [-- [PACMAN OPTIONS]]
Let us say you want to downgrade opera web browser to any available old version.

To do so, run:

$ sudo downgrade opera
This command will list all available versions of opera package (both new and old) from your local cache and remote mirror.

Sample output:

Available packages:

1.  opera-37.0.2178.43-1-x86_64.pkg.tar.xz (local)
2.  opera-37.0.2178.43-1-x86_64.pkg.tar.xz (remote)
3.  opera-37.0.2178.32-1-x86_64.pkg.tar.xz (remote)
4.  opera-36.0.2130.65-2-x86_64.pkg.tar.xz (remote)
5.  opera-36.0.2130.65-1-x86_64.pkg.tar.xz (remote)
6.  opera-36.0.2130.46-2-x86_64.pkg.tar.xz (remote)
7.  opera-36.0.2130.46-1-x86_64.pkg.tar.xz (remote)
8.  opera-36.0.2130.32-2-x86_64.pkg.tar.xz (remote)
9.  opera-36.0.2130.32-1-x86_64.pkg.tar.xz (remote)
10. opera-35.0.2066.92-1-x86_64.pkg.tar.xz (remote)
11. opera-35.0.2066.82-1-x86_64.pkg.tar.xz (remote)
12. opera-35.0.2066.68-1-x86_64.pkg.tar.xz (remote)
13. opera-35.0.2066.37-2-x86_64.pkg.tar.xz (remote)
14. opera-34.0.2036.50-1-x86_64.pkg.tar.xz (remote)
15. opera-34.0.2036.47-1-x86_64.pkg.tar.xz (remote)
16. opera-34.0.2036.25-1-x86_64.pkg.tar.xz (remote)
17. opera-33.0.1990.115-2-x86_64.pkg.tar.xz (remote)
18. opera-33.0.1990.115-1-x86_64.pkg.tar.xz (remote)
19. opera-33.0.1990.58-1-x86_64.pkg.tar.xz (remote)
20. opera-32.0.1948.69-1-x86_64.pkg.tar.xz (remote)
21. opera-32.0.1948.25-1-x86_64.pkg.tar.xz (remote)
22. opera-31.0.1889.174-1-x86_64.pkg.tar.xz (remote)
23. opera-31.0.1889.99-1-x86_64.pkg.tar.xz (remote)
24. opera-30.0.1835.125-1-x86_64.pkg.tar.xz (remote)
25. opera-30.0.1835.88-1-x86_64.pkg.tar.xz (remote)
26. opera-30.0.1835.59-1-x86_64.pkg.tar.xz (remote)
27. opera-30.0.1835.52-1-x86_64.pkg.tar.xz (remote)
28. opera-29.0.1795.60-1-x86_64.pkg.tar.xz (remote)
29. opera-29.0.1795.47-1-x86_64.pkg.tar.xz (remote)
30. opera-28.0.1750.51-1-x86_64.pkg.tar.xz (remote)
31. opera-28.0.1750.48-1-x86_64.pkg.tar.xz (remote)
32. opera-28.0.1750.40-1-x86_64.pkg.tar.xz (remote)
33. opera-27.0.1689.76-1-x86_64.pkg.tar.xz (remote)
34. opera-27.0.1689.69-1-x86_64.pkg.tar.xz (remote)
35. opera-27.0.1689.66-1-x86_64.pkg.tar.xz (remote)
36. opera-27.0.1689.54-2-x86_64.pkg.tar.xz (remote)
37. opera-27.0.1689.54-1-x86_64.pkg.tar.xz (remote)
38. opera-26.0.1656.60-1-x86_64.pkg.tar.xz (remote)
39. opera-26.0.1656.32-1-x86_64.pkg.tar.xz (remote)
40. opera-12.16.1860-2-x86_64.pkg.tar.xz (remote)
41. opera-12.16.1860-1-x86_64.pkg.tar.xz (remote)

select a package by number:
Just type the package number of your choice, and hit enter to install it.

That's it. The current installed package will be downgraded to the old version.


## Reference List {#reference-list}

1.  <https://ostechnix.com/downgrade-package-arch-linux/>
