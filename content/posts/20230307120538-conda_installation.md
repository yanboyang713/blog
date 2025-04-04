---
title: "Conda Installation"
draft: false
---

Installation details, please have a look the [Official Installation guide](https://conda.io/projects/conda/en/latest/user-guide/install/linux.html).

There are brief steps at the below.

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```


## [Arch Linux]({{< relref "20230220222636-arch_linux.md" >}}) {#arch-linux--20230220222636-arch-linux-dot-md}

```bash
paru -S miniconda3
```


### Set with your shell {#set-with-your-shell}

[Find out your current shell name]({{< relref "20230307121834-shell.md#find-out-your-current-shell-name" >}})


#### If your shell is Bash or a Bourne variant, enable conda for the current user with {#if-your-shell-is-bash-or-a-bourne-variant-enable-conda-for-the-current-user-with}

```bash
echo "[ -f /opt/miniconda3/etc/profile.d/conda.sh ] && source /opt/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
```

or, for all users, enable conda with

```bash
sudo ln -s /opt/miniconda3/etc/profile.d/conda.sh /etc/profile.d/conda.sh
```


## init shell {#init-shell}

To initialize your shell, run

```bash
conda init <SHELL_NAME>
```

Currently supported shells are:

-   bash
-   fish
-   tcsh
-   xonsh
-   zsh
-   powershell
