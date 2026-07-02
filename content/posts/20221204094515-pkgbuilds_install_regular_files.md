---
title: "PKGBUILDs install regular files"
tags: ["PKGBUILD"]
draft: false
---

It depends what folder contains. if it only contains regular files (non-executables):

```bash
install -d ${pkgdir}/opt/
install -m 644 ${srcdir}/${pkgname}-${pkgver}/folder/* ${pkgdir}/opt/
```

If it contains subdirectories:

```bash
install -d ${pkgdir}/opt/
cp -r ${srcdir}/${pkgname}-${pkgver}/folder/* ${pkgdir}/opt/
```


## Reference List {#reference-list}

1.  <https://bbs.archlinux.org/viewtopic.php?id=78476>
