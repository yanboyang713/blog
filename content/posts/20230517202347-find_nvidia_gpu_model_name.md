---
title: "Find NVIDIA GPU Model Name"
draft: false
---

[Nvidia]({{< relref "20230405140352-nvidia.md" >}})

please update your PCI ID database with:

```bash
sudo update-pciids
```

And use the following command in your terminal:

```bash
lspci -nn | grep '\[03'
```

You will see the model name of your graphic card. If it's ambiguous, you could search the PCI ID (something like [10de:11bc]) on the Internet for the corrent model name.
