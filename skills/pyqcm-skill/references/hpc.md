# Running pyqcm on an HPC cluster (Grappe IQ worked example)

This file documents **one specific cluster**, the Institut quantique compute cluster ("Grappe IQ") at
Universite de Sherbrooke, as a concrete worked example of running pyqcm in a module-based HPC
environment. If you are on a different cluster, the shape transfers (module environment, SLURM,
scratch-vs-home storage) but every name below will differ. Confirm the modules for your own cluster
rather than copying these.

Source: https://ccs-udes.github.io/hpc-iq/ (also available in English at
https://ccs-udes.github.io/hpc-iq/en/). Content below is summarised from the cluster documentation as
captured 2026-07-23. Check upstream before trusting version numbers or module versions, which drift.

The IQ cluster mirrors the Digital Research Alliance of Canada national clusters (Linux, `module`,
SLURM), so https://docs.alliancecan.ca applies too, and moving a calculation between them is cheap.

## Connecting

```bash
ssh <user>@hpc.iq.ccs.usherbrooke.ca
```

CCDB credentials plus Duo MFA. `ssh-copy-id` works and is worth doing. Note the cluster does **not**
use the SSH public keys registered in your CCDB account; you must copy your key explicitly. The login
node is `iv11`.

## Storage, and the rule that matters

| Location | Use for | Do not use for |
|---|---|---|
| `/home/$USER` | config, code, software you install | reading/writing research data in jobs |
| `/net/nfs-iq/data/$USER` | your research data, job IO | - |
| `/net/nfs-iq/data/def-$PROF` | group-shared research data | - |
| `$SLURM_TMPDIR` | per-job scratch on node-local disk, fastest IO | anything you need after the job ends |

`$HOME` has limited capacity and poor IO. Point pyqcm output at `/net/nfs-iq/data`, not at `$HOME`.
Research data storage is 190T, redundant, backed up daily and archived to tape weekly.
`$SLURM_TMPDIR` is deleted when the job finishes.

## Installing pyqcm on the cluster

**Precompiled package**, the fast path:

Do not use the precompiled package unless you know what you are doing. It is
a stale/un-maintained version.

**From source**, for a development version or to experiment with build options:

```bash
git clone https://github.com/pyqcm-project/pyqcm.git qcm_wed
cd qcm_wed
module load StdEnv/2023 gcc/12.3
module load python/3.11.5 scipy-stack/2025a
module load flexiblas/3.3.1 eigen/3.4.0 cuba/4.2.2 primme/3.2 nlopt/2.10.0
virtualenv --no-download $HOME/venv/qcm-dev
source $HOME/venv/qcm-dev/bin/activate
pip install --no-index --upgrade pip
export CMAKE_ARGS="-DEIGEN_HAMILTONIAN=1 -DWITH_PRIMME=1 -DBLA_VENDOR=FlexiBLAS -DPRIMME_DIR=$EBROOTPRIMME -DCUBA_DIR=$EBROOTCUBA -DWITH_GF_OPT_KERNEL=0"
pip install . --no-index
```

`--no-index` throughout: the cluster serves prebuilt wheels locally and you want those, not PyPI.

### Two traps, both load-bearing

- **Build against FlexiBLAS, not MKL directly.** GCC-compiled pyqcm linked against Intel MKL activates
  two OpenMP runtimes at once, and requesting `n` threads spawns `n * n`. This is a severe slowdown
  that reads like "the cluster is busy". `-DBLA_VENDOR=FlexiBLAS` avoids it. See "OpenMP threads"
  in `references/practice.md` for the thread-budget guidance that goes with it.
- **Compile on the node you will run on** if you use CPU-specific optimisation (`-march=native`,
  `-xHost`) or `-DWITH_GF_OPT_KERNEL=1`. A login-node build can be incompatible with some compute
  nodes and dies with "illegal instruction". Build inside an interactive job (`salloc`) in that case.

The pyqcm authors report poor performance with BLIS. On the IQ cluster, Intel MKL is used by default
through the FlexiBLAS interface, which is the combination you want.

## Submitting jobs

**Never run a calculation on the login node. Use `sbatch`, `salloc`, or `srun`.**

Default partition is `iq-main`:

```bash
#!/bin/bash
#SBATCH --job-name=my-job
#SBATCH --partition=iq-main
#SBATCH --cpus-per-task=8
```

- Public nodes: three nodes, 96 CPU cores and 500G memory each. Maximum walltime seven days.
- GPU: one node with two Nvidia A40, 48 cores, 500G. Request with `--gpus-per-node=nvidia_a40:2`,
  `--gpus-per-task`, or `--gres=gpu`.
- Contributed nodes: request the matching partition with `-p`. Combine with commas,
  `--partition=iq-main,iq-alice`; public nodes are preferred regardless of order. `--exclude=<node>`
  drops one.
- `iq-preempt` shares contributed nodes, available if your group contributes one. Jobs there are
  killed when the node owner needs it: `SIGTERM` plus one minute of grace, and `--requeue` puts the
  job back in the queue. Good for batches of short jobs and for checkpointing workloads.

Check available resources with `susage --legend`. Python packages available as local wheels come from
`avail_wheels`. Default environment is `StdEnv/2023`, optimisation target AVX2, default BLAS/LAPACK is
Intel MKL.

Set `OMP_NUM_THREADS` to half of `--cpus-per-task` on SLURM clusters. See `references/practice.md`.
