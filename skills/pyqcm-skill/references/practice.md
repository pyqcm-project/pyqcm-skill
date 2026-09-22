# Pyqcm Gotchas

A list of worth-knowing behaviours of [Pyqcm](https://github.com/pyqcm-project/pyqcm), mostly related to my daily usage of Cluster Dynamical Mean-Field Theory (CDMFT).

I will hopefully update this document on a daily basis in order to enhance the published [pyqcm-skill](https://github.com/pyqcm-project/pyqcm-skill) and be able to help my colleagues when needed.

> **Note**:
> Some entries below depend on the Pyqcm version and name it explicitly (e.g. "up to v2.26.x", "as of v2.29.x"). This skill ships no Pyqcm source and is not pinned to a release, so check which version is actually installed (`python3 -c "import pyqcm; print(pyqcm.__version__)"`) and what `$PYQCM_ROOT` is checked out at (`git -C $PYQCM_ROOT describe --tags`) before trusting a version-qualified claim. They are not necessarily the same.
>
> Some of what follows reflects choices made by the code owner (David Senechal) that could change in a future release. Those are flagged where they occur, so double-check rather than trusting them blindly.

## Installation

The most straight-forward way to install Pyqcm is by using `pip install .`. However, this command won't work unless those numerical libraries are properly installed on the machine:

- C++ compiler
- LAPACK, OpenBLAS
- Eigen
- HDF5
- OpenMP

On MacOS, the best way to install those libraries is by using the [Homebrew](https://brew.sh) package manager whereas on Linux, I am pretty sure you already have your personal favorite... On Institut Quantique's HPC nodes, the story is different, and I think that you should always refer to the updated(?) [documentation](https://ccs-udes.github.io/hpc-iq/en/). BUT, as for now, loading the following modules works for me:

```bash
$ module purge
$ module load StdEnv/2023 gcc Eigen scipy-stack nlopt cmake hdf5
```

About the Python environnement, I highly suggest that you use a *virtual environnement manager* (which is way less fancy than it sounds like). Most of the group members use [uv](https://github.com/astral-sh/uv), a blazing fast rust-based manager. A typical workflow for new virtual environnement looks like this:

```bash
$ uv venv pyqcm-venv
$ source ./pyqcm-venv/bin/activate
(pyqcm-venv) $ uv pip install <more dependencies>
```

Anyway, when installing Pyqcm with `pip install .`, the only thing that changes is the prefix uv: `uv pip install .`. Since the root directory of Pyqcm contains all the dependencies information in the pyproject.toml, you can install it inside a freshly created virtual environnement.

> **Tips and Tricks**:
> Add the `-e` flag (meaning *editable*) to your installation command makes the source library directory the executable version used by the environnement. It is convenient for developpment since you do not have to reinstall the library for every tiny changes you make **to the Python interface**.
> 
> Concretely, all of the files ./pyqcm/pyqcm/\*.py will be the ones used by your virtual environnement whereas without the `-e` flag, the files used will be copied in your virtual environnement (e.g. pyqcm-venv/lib/python3.x/site-packages/pyqcm/\*.py) and won't change unless you reinstall the library.

> **Note**:
> On other Digital Research Alliance clusters, expect a similar `StdEnv/2023`-style module environment, but confirm the exact module names for that cluster rather than assuming they match Grappe IQ. The full IQ recipe, including the `CMAKE_ARGS` line, lives in `references/hpc.md`.

## OpenMP threads

Some scripts/processes living in Pyqcm can be parallelized using the concept of threads, i.e. *shared-memory parallelization*. To understand what is a thread, let's start from the main CPU (Central Processing Unit):

- **MCP (Multi-Core Processor)**: Main chip, typically one per computer/motherboard. Its job is to coordinate and communicate with all the other components (RAM, SSD, etc.).
    - **CPU cores**: Physical central processing units present on the main chip. Typical number of CPU cores is between 8 to 16. This number determines how many tasks you can run simultaneously.
        - **Threads**: Sets of instruction to the MCP in order to distribute the load of one process across the CPU cores. Hence, threads share memory and only exist as a subset of the process that summoned them (e.g. A process could be a C++ executable that would distribute its load across a specified number of CPU cores through the usage of the OpenMP library).

> **Note**:
> On more exotic MCPs, the concept of *hyperthreading* also exist. Essentially, hyper-threads are virtual cores living in one CPU core that are used for intensive parallel computing.

When importing Pyqcm in a Python shell, you will always see this:

```bash
>>> import pyqcm
Number of OpenMP threads = x
>>>
```

meaning that Pyqcm has read the the global variable `OMP_NUM_THREADS` of your computer and printed its value. It is also the value Pyqcm intends to use for parallel computing within its C++ backend. For better performance (crucially needs some study) we typically set the variable to four or 8 threads:

```bash
$ export OMP_NUM_THREADS=4 (or 8)
```

depending on the CPU cores budget.

> **Tips & Tricks**:
> An empirical rule specific for HPC nodes is that the number of threads should never be greater than half the number of requested CPU cores. I agree that it doesn't make any sense, but the simulations run faster when respecting this rule. As an example, an SBATCH configuration that would respect this rule can look like this:
> 
> ```bash
> #!/bin/bash
> #SBATCH --job-name=<name of your job>
> #SBATCH --account=<your supervisor account>
> #SBATCH --time=<time of your job>
> #SBATCH --cpus-per-task=16
> #SBATCH --mem-per-cpu=<number of gigabytes per CPU cores>G
> #SBATCH --partition=<partition of your supervisor, if applicable>
> 
> export OMP_NUM_THREADS=8
> ```
> **Note**:
> The empirical half-the-cores rule above may have a concrete cause. If Pyqcm is compiled with GCC but linked against Intel MKL as the BLAS backend, two OpenMP runtimes are active at once (GCC's and Intel's), and requesting $n$ threads spawns $n\times n$ instead of $n$. Building against the FlexiBLAS interface (`-DBLA_VENDOR=FlexiBLAS` in `CMAKE_ARGS`) avoids it. See `references/hpc.md`.
>
> Separately, when searching the ground state across several sectors, `pyqcm.set_global_parameter("parallel_sectors", True)` parallelizes that search. Note the explicit `True` second argument.

## Target sectors

As said in the documentation of Pyqcm, the target sectors set by the user constrains the ED solver to subsets/subblocks of the hamiltonian in which search for the ground state.

There are three *quantum number* to set:

- Irreducible representation: R0, R1, R2, ... Rn where n is a number (in base 2) determined by the parity of a representation under x/y reflection.
- The particle number: N1, N2, N3, ...  where the maximum value is 2\*number of orbitals.
- The spin projection ($S=2S_{z}$): S0, S1, S-1, S2, ...where the number of fermions of you system dictates the available values.

> **Tips & Tricks**:
> Let's take an example to illustrate the irreducible representation number by considering $C_{2v}$ abelian point group, which character table is given by:

| $C_{2v}$ | $e$ | $C_2$ | $m_y$ | $m_x$ |
| -------- | --- | ----- | ----- | ----- |
| $A_1$    | 1   |  1    | 1     | 1     |
| $A_2$    | 1   | 1     | -1    | -1    |
| $B_1$    | 1   | -1    | 1     | -1    |
| $B_2$    | 1   | -1    | -1    | 1     |
> Here, according to the documentation, the following mapping would be made between the Rx notation and the irreducible representation:
>     - $A_1\longrightarrow\;$R==0==, $A_1$ is even under $m_y$ and $m_x$: 00===0==.
>     - $A_2\longrightarrow\;$R==3==, $A_2$ is odd under both $m_y$ and $m_x$: 11===3==.
>     - $B_1\longrightarrow\;$R==1==, $B_1$ is odd under $m_x$: 01===1==.
>     - $B_2\longrightarrow\;$R==2==, $B_2$ is odd under $m_y$: 10===2==.

As an example, an half-filled two site cluster with C2 point group generator would require me to set the following sectors:

```python3
model.set_target_sectors(["R0:N2:S0/R1:N2:S0"])
```

> **Tips & Tricks**:
> If you specify an odd number of fermions, you cannot set the spin projection to be 0 since fermions have $S_{z} = \pm\frac{1}{2}$. In other words, it has to be an odd number as well.

The `/` character separates the available sectors for the first cluster. Then, for multiple clusters, we need to add the comma as well:

```python3
model.set_target_sectors(["<sectors cluster 1>", "<sectors cluster 2>"])
```

Then, let's say that you are looping over some band parameter, e.g. the chemical potential. As you sweep, the number of particles in you system may change, and the targeted sectors become wrong. In this case, you have to ensure that you give ALL the sectors that you think the system will need in order to converge. If it is not evident, it is a good practice to give more than needed.
> **Note**:
> Two additions to the rules above.
>
> When an odd particle number forces a nonzero spin projection, declare **both signs**, e.g. `R0:N3:S-1/R0:N3:S1`, not just one of them.
>
> When `generators=...` is declared on a `cluster_model`, the Hilbert space splits into one sector per irreducible representation on top of the existing N/S sectors, and nothing guarantees the ground state sits in the trivial irrep R0. A d-wave anomalous bath coupling can favour a non-trivial one. Target all of them, e.g. `["R0:S0/R1:S0/R2:S0/R3:S0"]` for a $C_{2v}$-generated cluster.
>
> For Mott transitions specifically, see arXiv:2509.07931 (fig. 7) and SciPost Phys. Codebases 23 rather than reasoning from general Hubbard-model intuition.

## GS consistency

A so-called *consistency* test is ran at each CDMFT iteration. Essentially, it ensures that the ground state computed by the ED solver was found in the right symmetry sector. Recalling the nomenclature of the sectors from the previous gotcha `Rx:Nx:Sx` i.e. the irreducible representation, particle number and spin projection.

This test used to print a warning directly in the terminal for us to see it when monitoring the CDMFT simulation, but it has been moved to the output `cdmft.tsv` and `cdmft_iter.tsv` files under columns `ConsistencyCheck_x`. That being said, it is still possible to have it displayed in the terminal by setting:

```python3
pyqcm.warnings = True
```

in a Python script. It is quite important to be stated since a wrong symmetry sector essentially makes the converged solution unreliable and non-physical.
> **Note**:
> `pyqcm.warnings = True` restores the banner, but **the run still continues**. To make a failed check stop the run, pass `cdmft(..., check_ground_state=True)`, which raises `ValueError` instead of logging. It defaults to `False` and has been there since at least v2.26.x, so it is easy to miss.
>
> The check compares the wavefunction density against the Green-function density with a threshold of `1e-4`. The signed difference always lands in `ConsistencyCheck_{n}` regardless of either switch.
>
> Output file naming changed upstream: up to v2.26.x `cdmft()` took `file="cdmft.tsv"` and `iter_file="cdmft_iter.tsv"`; as of v2.29.x there is a single `file="cdmft"` prefix and `iter_file` is gone, so passing it raises `TypeError`. `vca()` changed the same way. Default output names are unchanged.

## Operators' naming convention

Pyqcm uses the underscore character `_` to label different clusters. Hence, when defining a new cluster operator of lattice operator or any name related to a Pyqcm object, you should avoid using this character.

In any case, Pyqcm will shout at you for using it, but I still suggest that you do not use this character when defining lattice models.
> **Note**:
> This fails loudly, not silently. If you ever see the convention violated and Pyqcm did *not* throw, do not trust the result: something else is wrong.

## Lattice vs Superlattice

When creating a `pyqcm.lattice_model()`, one has to give the argument `superlattice` as an iterable of three-dimensional vectors. The given vectors specify how to tile the given cluster(s) in space, spanning the studied crystal structure. Those vectors **cannot** contain non-integer values. As an example, tiling a one dimensional lattice with a four site cluster would lead to such superlattice vector:

```python3
superlattice=[
    [4, 0, 0],
]
```

By default, the `lattice` argument is set to `None`, but its real default value is:

```python3
lattice=[
    [1, 0, 0],
    [0, 1, 0],
    [0, 0, 1]
]
```

That means, if the studied lattice is not a line, square, a cube, then you must adjust the lattice vectors with a choice of basis through:

```python3
model.set_basis([[<basis vect 1>], [<basis vect 2>], [<basis vect 3>]])
```
> **Note**:
> Whenever lattice vectors are specified explicitly, call `model.set_basis()` right after, **even when their values are identical to the superlattice vectors**. Skipping it raises no error and does not break the physics or the self-consistency. What breaks silently is plotting: k-space figures (spectral function, Fermi surface) come out in the wrong basis. Check that plot axes match intent before trusting a k-space figure.

## Some Pyqcm global parameters

Among **all** Pyqcm's global parameters, here are the ones that I actually use in my scripts:

- `pyqcm.warnings = True`: 
    - **What**: Prints the GS consistency warnings in the terminal.
    - **When**: CDMFT calculations.
- `pyqcm.set_global_parameter('parallel_sectors')`: 
    - **What**: Uses openMP to parallelize the computation of the Green function structures across the different sectors (uses more memory).
    - **When**: CDMFT calculations.
- `pyqcm.set_global_parameter('kgrid_side', 64)`:
    - **What**: Number of wavevectors on each side of a fixed wavevector grid (used for every direction unless overridden by `set_wavevector_grid(nkx, nky, nkz))`.
    - **When**: Access more momentum-space resolution.
- `pyqcm.set_global_parameter('temperature', 0.01)`:
    - **What**: Temperature of the system.
    - **When**: Smoothing the sectors transitions when sweeping over a lattice parameter in CDMFT, i.e. studying one-site density as a function of the chemical potential (fig. 14 in ref[^1]).
## Some CDMFT parameters

Among the ~30 CDMFT class attributes, here are the ones that actually matter explaining and thinking about:

- `grid=frequency_grid(type='regular', specs=(2, 15, 5))`:
    - **What**: The imaginary frequency grid on which the CDMFT distance function is evaluated.
    - **Why**: It has proved itself to be the most stable integration grid over the past years. A lot of older/published results have been produced with a similar grid (or Matsubara) so comparison is closer.
- `iteration='fixed_point'`:
    - **What**: It is the outer loop convergence algorithm of CDMFT. Depending on a given criteria (`convergence=<criteria(s)>`), this algorithm determines if CDMFT has converged or not and decides how to drive the next iteration, e.g. suggest a new set of bath parameters.
    - **Why**: It has proved itself to be the most stable convergence algorithm over the past years. Especially when starting with  dummy values of variational parameters. However, it can be slower than 'broyden' when starting from a converged solution.
- `alpha=(0.3, 12)`:
    - **What**: Damping factor for the next iteration and the number of CDMFT iteration after which damping is toggled on, respectively. Concretely, these specific parameters mean: after twelve CDMFT iterations, each suggested point will be mixed with 30% of the previous one, i.e. $x_{n} = (\alpha - 1)F(x_{n-1}) + x_{n-1}$ where $F(x) = x - x_{\text{{minimized}}}$.
    - **Why**: Very robust for superconductivity. Not necessary in the normal state.
- `method='trf'`:
    - **What**: Stands for **Trusted Region Reflective**. It is our most efficient bath parameters optimization algorithm.
    - **Why**: The difference is that it transforms the hybridization function into a residual vector and translates the minimization into a least squares problem which solution is found from our own jacobian implementation. The jacobian is approximated by central finite differences where the infinitesimal step is defined through a global parameter: `pyqcm.global_parameter('cdmft_jacobian_delta', 1e-5)`. Note that the central finite differences is also implemented natively in Scipy: `scipy.optimize.least_squares(jac='3-point')`. It gives the same converged bath parameters, at least in the case of the four sites plaquette at half-filling in the normal state. And, using subbaths for the same system.
### Trusted Region Reflective

WIP.

## Symmetries via `bath-parametrizer`

Writing `generators=[]` and `bath_irrep` by hand is fiddly, because each generator row encodes both the cluster-site permutation and the phase every bath orbital picks up under that operation. The `bath-parametrizer` package derives them from a point group instead. It works with subbaths too, and if the cluster has no bath you simply set `bath_irrep=False`.

```python3
from bath_parametrizer.bath_parametrization import BathParametrizer

sites = [[0, 0, 0], [1, 0, 0], [0, 1, 0], [1, 1, 0]]
p = BathParametrizer(sites, "C2v")

generators = p.get_pyqcm_generators(8, "C2v")   # 8 bath orbitals, 2 per irrep
pyqcm.cluster_model(4, 8, generators=generators, bath_irrep=True)

gens = p.get_pyqcm_generators(8, "C2v", subbath={"nsb": 3, "irreps": "replica"})
```

> **Tips & Tricks**:
> `get_pyqcm_generators` returns a flat list when `nsb` is 1 and a dict keyed by subbath index when `nsb > 1`, so check which one you got. An empty dict means the point group has no generators defined, i.e. `bath_irrep` is unavailable and you fall back to `bath_irrep=False`.
>
> Bath-phase blocks follow the SALC label order, so declare your `eb{i}`/`tb{i}` in that same order. A mismatch silently couples the wrong bath orbitals, which is the nastiest failure mode here since nothing complains.

Setup and the full API are in `references/bath-parametrizer.md`.
## `hopping_operators` amplitude convention

Write `hopping_operators` with amplitude `-1`. It is a sign convention matching how the group writes the Hamiltonian, not a code requirement. Using `+1` runs fine but flips the sign of the hopping term, so flag it if you meet it in someone else's script before interpreting the physics.
## Bath parameter naming and starting values

Bath parameters are named `ebi` for a bath energy and `tbi` for a cluster-bath hopping, `i` indexing the bath sites. For starting values, alternate `+1`/`-1` across the bath energies and use `0.5` for the cluster-bath hoppings.

`convergence='self-energy'` is arguably more physically motivated than the default `convergence='parameters'`, though both usually work. If `method='trf'` struggles, fall back to `'bobyqa'` with tight tolerances: `accur_bath=1e-6, accur_dist=1e-12`, or `accur_bath=1e-5, accur_dist=1e-10`.

> **Note**:
> The default frequency grid changed upstream. `frequency_grid` in `pyqcm/cdmft.py` was `grid_type="legendre", specs=(1, 10, 5, 10, 5)` up to v2.26.x and is `grid_type="regular", specs=(10, 50, 10)` as of v2.29.x. A script relying on the old default silently changes grid on upgrade, so pin it explicitly when comparing against older numbers.
>
> Senechal's bath-optimization paper (arXiv:1005.1685) is the study behind the weight-function choice. A weight proportional to `Tr Sigma^2` works best overall, especially for tracking a U-driven Mott transition, while `W = 1/omega` does badly in metallic phases where the self-energy is already small at low frequency. It also suggests `beta = 100/t` for the frequency spacing. It is a 2010 paper on a narrower set of models than we run now, so treat it as the strongest existing guidance rather than a closed case.
## Grounding a named material in the literature

When a script targets a real compound rather than a generic toy Hubbard model, the parameter choices, the cluster and bath geometry, and the expected order-parameter behaviour are all physics claims, even though what you are editing is a Python script. Check the "Grounding claims in the literature" section of `references/physics.md` and the relevant papers in `references/research/CITATIONS.md` before settling them.
## Model-construction landmines

Met while building `pyqcm-w90-builder` (2026-07), all verified against the Pyqcm source. Only the orbital-label point is Wannier90-specific.

- `segment_dispersion()` returns `None`, since it is a plotting function. For the eigenvalue array, do `k,_,_ = pyqcm.wavevector_path(nk, path)` then `e = instance.dispersion(k)`.
- Only one `lattice_model` can exist per process. Building several of them, in pytest for instance, needs `pyqcm.reset_model()` in between. Module-level models also need `importlib.reload(module)` after the reset, otherwise you hit a "cluster model name already used" collision.
- `set_basis` enters the k-phase, so two encodings differing only by `set_basis` give different bands at the same reduced k. They are the same band structure reparametrized. Compare two encodings inside a common `set_basis` frame. Orbital labels, on the other hand, are pure integer combinatorics and do not depend on `set_basis`.
- The orbital label follows the site listing order, so list your cluster sites in the orbital order you intend.
- `cluster_averages()` returns only the cluster-model operators, as `{name: (avg, var)}`. Lattice operators are not in there. `<epsX>` is the orbital occupation. The `_1_ave` column of `cdmft.tsv` is the self-consistent-loop average and differs from a single-shot re-solve, so compare two encodings by re-solving both rather than by comparing a re-solve against a stored `_ave`.
- `set_params_from_file()` needs the parameter set to exist already: call `set_parameters(...)` once, then `set_params_from_file(tsv, n=-1)`.
## Tied parameters must stay out of `varia`

Dependent parameters are written `X = c*Y` in the parameter string, e.g. `eb1_3 = 1*eb1_1`. Use them to impose a relation that the target sectors and the cluster point group do not already enforce: tying sites or bath orbitals that symmetry makes equivalent, imposing a pairing phase with `-1*`, or locking a ratio the model definition fixes. The tie follows the master through the whole optimization, not just at seed time.

Put only the master names in `varia`. Listing a dependent parameter there hands the optimizer a degree of freedom that the constraint immediately overwrites. Build the `varia` list and the tie string from the same code path so they cannot drift apart.
## Converged bath parameters are gauge-dependent

Bath parameters are not observables. The parametrization is many-to-one onto physical solutions, so two converged runs can print visibly different `eb`/`tb`/`db` tables and still be the same solution. Read as physics, a gauge move looks like a discovery: an orbital that decoupled, a hybridization that changed sign, a level that crossed mid-sweep. Nothing in the CDMFT distance function prefers one branch over another, so a warm start, a reordered bath or a jittered seed can move you between them silently.

Which redundancies exist depends only on the channels the bath declares, not on the model or the material:

| Redundancy | Acts on | Sends | Present when |
| ---------- | ------- | ----- | ------------ |
| Phase / sign | one orbital, `c_b -> -c_b` | `(eb, tb, db) -> (eb, -tb, -db)` | always |
| Permutation | two orbitals in the same symmetry class | swaps their whole `(eb, tb, db)` triple | two or more equivalent orbitals |
| Particle-hole flip | one orbital, `c_b -> c_b^dag` | `(eb, tb, db) -> (-eb, db, tb)` | normal and anomalous channels both exist |

> **Tips & Tricks**:
> The sign redundancy means the sign of an individual `tb_i` carries no information by itself: only relative signs within an orbital do. The permutation one is why bath orbitals seem to trade places discontinuously along a continuation sweep.
