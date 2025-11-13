---
title: "UVA CS Computing Environment"
date: 2025-08-28
draft: false
---

[University of Virginia]({{< relref "2025-04-11-013919-university_of_virginia.md" >}})


## UVA CS Project Directories {#uva-cs-project-directories}

<https://www.cs.virginia.edu/computing/doku.php?id=project_directories>

Project directory /p/nmg5g


### attach CS project dir to Arch {#attach-cs-project-dir-to-arch}


#### Install SMB/CIFS tools {#install-smb-cifs-tools}

```bash
sudo pacman -Syu --needed cifs-utils gvfs-smb smbclient keyutils
```


### [Proxmox’s CIFS storage]({{< relref "2025-09-28-163711-proxmox_s_cifs_storage.md" >}}) {#proxmox-s-cifs-storage--2025-09-28-163711-proxmox-s-cifs-storage-dot-md}


## UVA CS [Slurm]({{< relref "2023-12-10-182914-slurm.md" >}}) cluster {#uva-cs-slurm--2023-12-10-182914-slurm-dot-md--cluster}

<https://www.cs.virginia.edu/computing/doku.php?id=compute_slurm#resource_limits>


### Create a project-scoped env {#create-a-project-scoped-env}

Pick a path under your project dir (example: /p/nmg5g/condaenvs/py312):

```bash
# on a login node (portal) or short salloc
module load miniforge
mkdir -p /p/nmg5g/condaenvs

# create the env *by path* so it lives in /p
conda create -y --prefix /p/nmg5g/condaenvs/py312 python=3.12
```

To use it interactively:

```bash
module load miniforge
# make 'conda activate' available in non-interactive shells
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /p/nmg5g/condaenvs/py312
python -V
```


### Keep caches out of /u (optional but recommended) {#keep-caches-out-of-u--optional-but-recommended}

By default, Conda’s package cache and named envs live in your home. Redirect them to /p with a .condarc:

```bash
rhe9cf@portal03:~$ cd
rhe9cf@portal03:~$ pwd
/u/rhe9cf
```

```bash
# create/edit ~/.condarc
cat >> ~/.condarc <<'YAML'
envs_dirs:
  - /p/nmg5g/conda/envs
pkgs_dirs:
  - /p/nmg5g/conda/pkgs
YAML

mkdir -p /p/nmg5g/conda/envs /p/nmg5g/conda/pkgs
```

Conda reads .condarc to set envs_dirs and pkgs_dirs. You can also use conda config --add envs_dirs ... / --add pkgs_dirs ....


### Use the env in Slurm jobs {#use-the-env-in-slurm-jobs}

create hello.py

```python
import sys, multiprocessing as mp
print("Python:", sys.version)
print("CPUs visible:", mp.cpu_count())
```

create conda_example.slurm

```file
#!/bin/bash
#SBATCH -p cpu
#SBATCH --cpus-per-task=4
#SBATCH --mem=8G
#SBATCH -t 01:00:00
#SBATCH -J conda-demo
#SBATCH --output=slurm-%A.out

set -euo pipefail
module --ignore_cache purge
module load miniforge

# make `conda activate` work in batch shells
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /p/nmg5g/condaenvs/py312

echo "Python: $(python -V) @ $(which python)"
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# use heredoc with the *activated* interpreter
srun python hello.py
```

run:

```bash
sbatch conda_example.slurm
```

```bash
squeue -u rhe9cf
```


## Reference List {#reference-list}

1.  <https://www.cs.virginia.edu/computing/doku.php?id=start>
