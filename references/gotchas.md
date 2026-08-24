# Gotchas and hard-won lessons

Lessons about pyqcm's sharp edges that aren't documented anywhere else — conventions this specific
research group relies on, mistakes made before, build quirks on this machine, things that look right
but aren't. Read this before starting scripting/physics/contributing work; add to it whenever a
session turns up a new one.

Sourced from Antoine de Lagrave's own notes (2026-07-22). Some of this reflects choices made by the
code owner (David Sénéchal) that could change in a future pyqcm release — where that's the case, it's
flagged below so a future reader knows to double-check rather than trust it blindly.

Entries whose behavior depends on the pyqcm version name the versions explicitly (e.g. "up to
v2.26.x" / "as of v2.29.x"). The `pyqcm/` submodule here is pinned, so check what it actually points
at (`git submodule status`) before trusting a version-qualified claim against the pyqcm you have
installed — they are not necessarily the same.

## Format for new entries

```
## <short title>

What: <the surprising behavior or mistake, stated plainly>
Why it happens: <the underlying reason, if known>
What to do instead: <the fix or the correct convention>
```

Keep entries short and concrete — this file is meant to be skimmed, not read end to end each time.

## Installing dependencies

What: `pip install .` fails or behaves oddly if the compiler toolchain and numerical libraries
(C++ compiler, LAPACK, OpenBLAS, Eigen, HDF5, OpenMP) aren't in place first, and the fix differs by
where you're installing.
Why it happens: pyqcm's build (CMake + nanobind, see `references/build.md`) links against these at
compile time; there's no bundled fallback.
What to do instead:
- **Personal machine (macOS):** install [Homebrew](https://brew.sh) first, then use it for the
  compiler/LAPACK/OpenBLAS/Eigen/HDF5 stack. Use a Python venv manager for the rest — `uv` is the
  group's preference (fast), `poetry` also works.
- **Institut Quantique cluster ("Grappe IQ"):** don't guess module names — the exact `module load`
  sequence and CMake flags for both the precompiled package and a from-source dev install are
  documented at `references/hpc-iq/software/pyqcm.rst.txt` (as of that doc: `StdEnv/2023`,
  `python/3.11.5`, `scipy-stack`, `flexiblas`, `eigen`, `cuba`, `primme`, `nlopt` — but read the file
  itself rather than relying on this list, it may drift). **That file's `git clone` command is stale**
  — it points at the old `bitbucket.org/dsenechQCM/qcm_wed` location. pyqcm now lives at
  `github.com/pyqcm-project/pyqcm` (public org, as of 2026-07-23) — use that URL instead for a
  from-source dev install; the module-load/CMake instructions in the doc otherwise still apply.
- **Compute Canada / Digital Research Alliance clusters generally:** expect a similar `StdEnv/2023`-style
  module environment, but confirm the exact modules for that specific cluster rather than assuming
  they match the IQ grappe.

## OpenMP thread count

What: Setting `OMP_NUM_THREADS` too high makes pyqcm calculations dramatically ("crazy") slower, not
faster.
Why it happens: two distinct causes, depending on where you're running.
- **General (any machine):** you're oversubscribing — running more OpenMP threads than physical
  cores available for you to use leads to contention, not speedup.
- **Grappe IQ specifically (and possibly other clusters with the same BLAS setup):** if pyqcm was
  compiled with GCC but linked against Intel MKL as the BLAS backend, *two separate OpenMP runtimes*
  (GCC's and Intel's) are active at once. Requesting `n` threads then spawns `n × n` threads instead
  of `n` — see `references/hpc-iq/software/pyqcm.rst.txt` for the official writeup. This is a much
  worse slowdown than ordinary oversubscription and is easy to misdiagnose as "the cluster is busy."
What to do instead:
- Personal computer: the automatic default (usually the full thread count) works fine in practice.
  There's no general need to halve it — just be mindful of not overloading the machine if you want
  to keep using it for other things while a calculation runs.
- Alliance/SLURM clusters generally: set `OMP_NUM_THREADS` to half the requested `--cpus-per-task`.
  Empirically verified, not yet fixed in the code.
- Grappe IQ specifically: the double-OpenMP problem is avoided by compiling pyqcm against the
  **FlexiBLAS** interface rather than linking MKL directly (`BLA_VENDOR=FlexiBLAS` in `CMAKE_ARGS`) —
  see the dev-install command in `references/hpc-iq/software/pyqcm.rst.txt`.
- When searching the ground state across multiple sectors (`model.set_target_sectors(...)`), also
  set `pyqcm.set_global_parameter("parallel_sectors", True)` so sectors are searched in parallel
  rather than serially — a separate, additive speedup from the thread-count question above.

## Symmetries via `bath-parametrizer`

What: Defining `generators=[]` and `bath_irrep` by hand for a cluster model (to exploit point-group
symmetry and speed up ED) is fiddly and error-prone to derive manually.
Why it happens: the generator list encodes both the cluster-site permutation and, when a bath is
present, the phase each bath orbital picks up under each group operation — not something you want to
re-derive by hand per cluster geometry.
What to do instead: use the **bath-parametrizer** package to generate the generators list
automatically from a point group. If the cluster has no bath, set `bath_irrep=False`; if it does, the
same package can also symmetry-parametrize the bath couplings. It also works combined with the
subbaths method.

```python
from bath_parametrizer.bath_parametrization import BathParametrizer

sites = [[0, 0, 0], [1, 0, 0], [0, 1, 0], [1, 1, 0]]   # list or ndarray both fine
p = BathParametrizer(sites, "C2v")

# Vanilla (non-SB-CDMFT) mixed bath: 8 bath orbitals, 2 per C2v irrep.
generators = p.get_pyqcm_generators(8, "C2v")           # flat list, ready to pass
pyqcm.cluster_model(4, 8, generators=generators, bath_irrep=True)

# Subbath (SB-CDMFT) variant: one generators list per subbath.
gens = p.get_pyqcm_generators(8, "C2v", subbath={"nsb": 3, "irreps": "replica"})  # {1: [...], 2: [...], 3: [...]}
```

Key API points:
- `get_pyqcm_generators(n_baths, abelian_pg, subbath=None, linked_sites=None)` is the one-call path to
  `cluster_model(generators=..., bath_irrep=True)`. It returns a **flat list** when `subbath["nsb"]`
  is 1 (the default) and a **dict keyed by 1-based subbath index** when `nsb > 1` — check which you
  got before passing it along.
- It returns an **empty dict** if the point group has no defined generators (non-abelian/unsupported),
  which is the signal that `bath_irrep` symmetry can't be used at all — fall back to `bath_irrep=False`.
- Bath-phase blocks follow the SALC label order from `get_hybridization_links` (character-table order,
  or `irrep_1`, `irrep_2`, ... when an irrep has multiplicity > 1). **Declare `eb{i}`/`tb{i}` bath
  parameters in that same order** — a mismatch here silently couples the wrong bath orbitals.
- `subbath["irreps"]` accepts `"replica"` (default) and `"unique"`; `"custom"`/`"mixed"` raise
  `NotImplementedError`. `linked_sites` must be a union of point-group orbits or you get a `ValueError`.
- Site positions are coerced with `np.asarray(..., dtype=float)`, so plain nested lists work (this was
  a bug until Aug 2026 — older checkouts require an explicit `np.ndarray`).

Status: it's a proper submodule of this skill repo now, at
`references/pyqcm-bath-parametrizer/` (source in `bath_parametrizer/bath_parametrization.py` and
`bath_parametrizer/point_groups.py`), pulled from
https://github.com/antoinedelagrave/pyqcm-bath-parametrizer. Import as
`from bath_parametrizer.bath_parametrization import BathParametrizer`. Supported point groups: `Cs`,
`C2`, `C2v`, `C3`, `C3v`, `C4`, `C4v`, `C6`, `C6v`. Note the submodule's own `README.md` is stale — it
shows a `src/bath_parametrization/` layout and a `from bath_parametrization import ...` import that
don't match the actual package; trust the paths above (and `tests/test_bath_parametrization.py` for
worked examples), not the README.

## Lattice vs. superlattice vectors

What: Lattice models require specifying both superlattice vectors and lattice vectors — usually
identical, but not always (e.g. non-trivial superlattice choices, or ordered/broken-symmetry phases
where the physical unit cell is larger than the cluster tiling would suggest). **Whenever lattice
vectors are specified explicitly** (as a separate call after the superlattice vectors), **even if
their values end up equal to the superlattice vectors**, a matching `model.set_basis()` call is
needed too.
Why it happens: pyqcm doesn't infer one from the other, and doesn't infer the plotting basis from
either. Skipping `set_basis()` in this situation doesn't raise an error or break the physics/
self-consistency itself — the calculation still runs and converges correctly. What breaks silently is
**plotting**: k-space plots (spectral function, Fermi surface, etc.) come out in the wrong units/basis
because the plotting code has no other way to know which basis to render in.
What to do instead: **there is no crash or exception if this is skipped — you get silently wrong plot
units, not wrong physics.** Whenever lattice vectors are specified explicitly, call
`model.set_basis()` right after, even if the lattice and superlattice vectors happen to be identical,
and double check plot axes match intent before trusting a k-space figure.

## `hopping_operators` amplitude convention

What: By convention, this group writes `hopping_operators` with amplitude `-1`.
Why it happens: this matches how the Hubbard/Anderson model Hamiltonian is conventionally written
down in the group's papers and notes — it's a sign convention, not a code requirement.
What to do instead: use `-1` to stay consistent with the group's convention and with
`references/physics.md`. Using `+1` will still run without error, but flips the sign of the hopping
term relative to the usual convention — worth calling out explicitly if you see it in someone else's
script so the physics interpretation doesn't get scrambled.

## Bath parameter naming, starting values, and solver choices

**Status: this whole entry is current best practice as of 2026-07-22, not settled fact.** The group is
still actively benchmarking some of these choices, so treat it as "what's worked so far" and check
with Antoine if it's been a while.

What: A cluster of related conventions around bath parametrization and CDMFT convergence:
- Bath parameter names: `ebi` (bath energy) and `tbi` (cluster-bath hopping), `i` indexing bath sites.
- Starting values: alternate `+1`/`-1` across bath energies, `0.5` for cluster-bath hoppings.
- CDMFT `iteration` option: default is `'broyden'` (faster once a nearby converged solution already
  exists, e.g. when looping over a parameter via `controlled_loop()` in `pyqcm/_loop.py`), but
  `'fixed_point'` tends to do better when no converged solution has been found yet.
- CDMFT `convergence` option: `convergence='self-energy'` is arguably more physically motivated than
  the default `convergence='parameters'`, but both usually work in practice.
- Minimization `method`: `'trf'` is currently the best default; if it struggles, fall back to
  `'bobyqa'` with very tight tolerances (`accur_bath=1e-6, accur_dist=1e-12` or
  `accur_bath=1e-5, accur_dist=1e-10`).
- The imaginary-frequency grid used for the CDMFT distance function (`src_qcm/CPT.cpp` around line
  519). **The default changed upstream:** `frequency_grid` in `pyqcm/cdmft.py` was
  `grid_type="legendre", specs=(1, 10, 5, 10, 5)` up to v2.26.x and is
  `grid_type="regular", specs=(10, 50, 10)` as of v2.29.x (commit `6813034`). A run that relied on the
  old default silently changes grid on upgrade, so pin `grid_type` explicitly rather than inheriting
  it if you are comparing against older numbers. Sénéchal's bath-optimization paper (`references/research/quantum_cluster_methods/1005.1685v1.txt`) is the detailed
  study behind this choice, benchmarking several weight functions `W(omega)` against Potthoff's
  self-energy functional approach (treated as the reference "best possible" bath). Its findings: a
  weight proportional to `Tr Sigma^2` is the most successful overall, especially for tracking a
  U-driven Mott transition; a weight that over-emphasizes low frequencies (`W = 1/omega`) does badly
  in metallic/weak-gap phases where the self-energy is already small at low frequency; a sharp
  frequency cutoff is more adequate specifically in the overdoped/small-gap region. It also gives a
  concrete rule of thumb for the fictitious inverse temperature setting the frequency spacing:
  `beta = 100/t` was sufficient in that paper's benchmarks (20-200/t was the range actually tried).
  Treat this as the strongest existing guidance on the grid/weight question, not a closed case — it's
  a 2010 paper on a narrower set of models than this group now runs, so if the choice matters for a
  specific result, still confirm current practice with Antoine rather than assuming it's
  unconditionally settled.
Why it happens: these are empirical choices refined through this group's usage, not documented
defaults elsewhere.
What to do instead: start from the above as sensible defaults, but don't treat them as immutable —
the frequency-grid/weight-function choice in particular has real physics behind it (see paper above),
not just trial and error, but it's still worth a sanity check against current group practice.

## Operator naming: no underscores

What: Operator names must not contain `_` (underscore).
Why it happens: pyqcm's C++ layer uses `_` internally as a separator to determine which cluster an
operator belongs to.
What to do instead: **this fails loudly, not silently** — the C++ layer throws an error if you name an
operator with an underscore. If you ever see this convention violated and the code *didn't* throw,
don't trust the result — something is off and needs investigating rather than being written off as a
naming quirk.

## Target sectors when sweeping a parameter to *probe* (not fix) a transition

What: When sweeping a parameter like chemical potential specifically to probe a Mott transition (as
opposed to studying a single fixed filling), the ground-state particle number `N` is expected to
change across the sweep — e.g. Fig. 7 of the subbath paper (`references/research/quantum_cluster_methods/2509.07931v2.txt`)
tracks density `n` vs. `μ` exactly this way, with `n` moving continuously except for a jump/plateau at
the Mott transition itself. A `model.set_target_sectors(...)` call listing only a single `N` will not
track this: the solver only ever searches the sector(s) you declare, so fixing `N` throughout the
sweep silently prevents the solver from ever finding the filling change that's the whole point of the
sweep.
Why it happens: sector search in pyqcm is not automatic — it only ever considers the sectors it's told
to. Nothing warns you that the true ground state might live outside your declared list; a run stays
"converged" while quietly reporting the wrong filling.
What to do instead:
- Declare multiple candidate `N` sectors spanning the range of fillings the sweep might visit (e.g.
  `N3, N4, N5` around half-filling for a 4-site cluster), not just the value expected at one end.
- For any sector with an **odd** total particle number `N`, you must also specify a nonzero spin
  sector — an odd number of electrons cannot produce total spin projection zero. Use both signs, e.g.
  `R0:N3:S-1/R0:N3:S1`, rather than `R0:N3:S0` (which is not a valid target and would either error or
  silently mean something other than intended).
- Mott transitions specifically are discussed in the subbath paper (`references/research/
  quantum_cluster_methods/2509.07931v2.txt`, Fig. 7) and in the SciPost pyqcm codebase paper
  (`references/research/quantum_cluster_methods/SciPostPhysCodeb_23.txt`, around the 1D Hubbard Mott gap example) — check those before reasoning
  about expected sector/filling behavior from general Hubbard-model intuition alone.

## Target sectors when point-group generators are declared

What: Declaring `generators=...` on a `cluster_model` (see "Symmetries via `bath-parametrizer`" above)
splits the cluster's Hilbert space into one sector per irrep of the point group (`R0`, `R1`, ... —
pyqcm's `R` label), on top of whatever `N`/`S` sectors already existed. The ground state can land in
*any* of these irrep sectors, not just `R0`.
Why it happens: same root cause as the `N`-sector gotcha above — sector search is not automatic, it
only considers what's declared. A point group with `k` one-dimensional irreps (e.g. `C2v`, abelian,
`k=4`: `A1,A2,B1,B2`) gives `k` irrep sectors, and nothing about the physics guarantees the trivial
irrep (`R0`) holds the ground state — e.g. a d-wave anomalous bath coupling can favor a
non-trivial irrep.
What to do instead: target all irrep sectors the declared point group produces (e.g.
`["R0:S0/R1:S0/R2:S0/R3:S0", ...]` for a `C2v`-generated cluster with 4 irreps), not just `R0:S0`, the
same way the `N`-sector gotcha requires spanning the filling range rather than fixing one value.

## Grounding a named material/system in the literature (not just "physics" tasks)

What: A session can be entirely "script work" by the SKILL.md job classification (bath symmetry,
CDMFT mechanics, sweep logic) while quietly also making physics-grounding decisions — model
parameters, cluster/bath geometry, expected order-parameter behavior — without ever opening
`references/physics.md`, because that file is only gated in for the "interpret physics results" job.
Why it happens: `SKILL.md`'s routing table is a single-job classification ("figure out which one the
user needs, read the matching file") rather than a set of independently-triggered checks — a script
task never trips the physics-file read even when it involves a real material with existing literature
under `references/research/`.
What to do instead: whenever a script targets a specific named physical system (a real material or
compound, not a generic toy Hubbard model), check `references/physics.md`'s "Grounding claims in the
literature" section and the relevant `references/research/<topic>/` papers for that system
*regardless* of whether the session otherwise reads as pure script/mechanics work — parameter choices
and expected qualitative behavior (order-parameter shape, competing orders, known discontinuities) are
physics claims even when the actual edit is a Python script.

## GS consistency test visibility in CDMFT

What: The ground-state consistency check that pyqcm runs at each CDMFT iteration used to print to the
terminal; it was made silent by the code owner (David Sénéchal), and now only shows up as the
`ConsistencyCheck_{n}` column of the CDMFT output files, not in live terminal output — even though the
simulation will still happily converge if this check fails.
Why it happens: a recent upstream change, made silent by design (the code owner appears to prefer
checking the output files after the fact rather than watching it live). Antoine's own habit is to
watch the terminal for the first several iterations of a launched script as a sanity check, and this
change makes it very easy to forget the check exists at all — since nothing on-screen tells you
it's missing.
What to do instead: **this test is essential — it verifies the ground state search is happening in
the right symmetry sector.** If it fails, it means the solver converged to a state outside your
declared sectors, and the physics is not trustworthy even though the run "succeeded." Two independent
switches control it (`model_instance.GS_consistency`, `pyqcm/__init__.py`), and you usually want both:

- `pyqcm.warnings = True` restores the banner in the terminal. **The run still continues.**
- `cdmft(..., check_ground_state=True)` makes a failure `raise ValueError` instead of being logged.
  Default is `False`, so by default nothing stops a run whose ground state is inconsistent. This
  argument is not new — it has been there since at least v2.26.x and is simply easy to miss.

The comparison is the wavefunction density against the Green-function density, with a `threshold` of
`1e-4`; the signed difference is always written to `props["ConsistencyCheck_{n}"]` (one per system)
regardless of either switch, which is what reaches the output files.

Note on those files: **the CDMFT/VCA output naming changed upstream.** Up to v2.26.x, `cdmft()` took
two separate arguments, `file="cdmft.tsv"` and `iter_file="cdmft_iter.tsv"`. As of v2.29.x there is a
single `file="cdmft"` **prefix** (no extension) and the iteration file is derived as
`<prefix>_iter.tsv` — `iter_file` is gone as an argument, so passing it is now a `TypeError`. `vca()`
changed the same way (`file="vca.tsv"` to `file="vca"`). Default output names are unchanged
(`cdmft.tsv`, `cdmft_iter.tsv`), so only scripts that passed these explicitly are affected.

## Model-construction and encoding landmines

Hit while building `pyqcm-w90-builder`'s SC-capable physical encoding (2026-07); all verified against
pyqcm source. Only the last bullet is Wannier90-specific — the rest apply to any pyqcm model.

- **`segment_dispersion()` returns `None`.** It is a plotting function (calls `plt.show()`). To get
  the eigenvalue array, replicate its internals: `k,_,_ = pyqcm.wavevector_path(nk, path)` then
  `e = instance.dispersion(k)` (returns `ndarray(nk, dimGF_red)`). `wavevector_path`'s `shape` arg
  accepts a `.tsv` k-path filename.
- **One `lattice_model` per process (hard singleton).** `lattice_model.__init__` raises
  "Only one lattice model can be defined at a time!" To build several models in one process (e.g.
  pytest), call `pyqcm.reset_model()` between them — it does a full C++ (`qcm.great_reset()`) +
  Python reset (clears `lattice_model.defined` and `cluster_model_names`). Module-level models
  (built at import) need `importlib.reload(module)` *after* a reset to rebuild cleanly, else you get
  a "cluster model name already used" collision.
- **`set_basis` enters the k-phase, so dispersion eigenvalues depend on it.** Two encodings that
  differ only by `set_basis` give *different* bands at the same reduced input-k (the input k maps to
  different physical k). They are the *same* band structure reparametrized. To check band
  equivalence of two encodings, evaluate both in a *common* `set_basis` frame. Orbital *labels*,
  by contrast, are pure integer combinatorics (position cosets mod the integer lattice,
  `src_qcm/lattice_model.cpp:93-102`) and are `set_basis`-independent.
- **Orbital label = coset-appearance order = site listing order** (when all cosets are distinct). So
  listing cluster sites in the intended orbital order keeps pyqcm indices 1..N aligned with your
  `labels`/interaction operators even after changing positions.
- **`cluster_averages()` returns only cluster-model operators**, as `{name: (avg, var)}` per system
  (`sys=` arg). Lattice operators (density-wave-style `n*` readouts, `Vdc`, `U`, `J`) are not in it.
  `<epsX>` (the on-site number operator) *is* the orbital occupation. The `_1_ave` column in the CDMFT
  solution file (`cdmft.tsv` by default, `<prefix>.tsv` in general — see the naming note in the GS
  consistency entry) is the self-consistent-loop cluster average and differs from a single-shot
  re-solve of the converged parameters — to compare two encodings, re-solve both and compare, don't
  compare a re-solve to the stored `_ave`. Unrelated but adjacent: `instance.averages()` appends to
  `averages.tsv` unless you pass `file=None`. The signature is unchanged across versions, but pyqcm's
  own internal callers started passing `file=None` in v2.29.x to cut file clutter, so that file may
  simply be quieter than you remember.
- **`set_params_from_file()` needs the parameter set already defined** — call `set_parameters(...)`
  once first (to declare/instantiate all params), then `set_params_from_file(tsv, n=-1)` overwrites
  with the converged row.
- **`.win` parsing is not uniform across materials.** `unit_cell_cart` may carry a leading units
  line (`ang`/`bohr`); atoms may be `atoms_cart` or `atoms_frac` (fractional → convert with the
  cell); structural atoms without a `projections` entry (Ba/Y in YBCO) must be filtered. Wannier
  centres count can exceed the pyqcm subspace size (YBCO: 49 centres, 36-orbital model).

## One-body / band checks on a model with a bath and a point group

Hit while writing a U=0 validation script for a two-cluster bilayer Emery model. Four traps in a row,
none of which are in the docs, all of which look like bugs in your own script.

- **`dispersion()` is a `model_instance` method, not a `lattice_model` one.** You still have to build
  `pyqcm.model_instance(model)` to reach it; `lattice_model` has no `dispersion`/`tk`, so the
  `AttributeError` reads as if the API changed when it didn't. (Same for `tk()` and `epsilon()`.)
  **Whether that forces an ED solve is version-dependent.** Up to v2.26.x, `QCM::dispersion` built the
  cluster Green function first, so target sectors and a full parameter set were mandatory before you
  could look at `t(k)`. As of v2.29.x it calls `bare_epsilon` and diagonalizes directly — a pure
  one-body path, no ED (`src_qcm/QCM.cpp`, `dispersion`; upstream commit `6813034`, "functions that
  compute the dispersion relation without performing ED first"). Source-verified, not yet
  runtime-verified: if a one-body check still demands solvable clusters on a recent pyqcm, that is
  worth reporting upstream rather than working around.
- **`U=0` *exactly* breaks a cluster that declares `generators=...`.** pyqcm routes a fully
  non-interacting cluster to `one_body_solve()`, which throws
  `The symmetry group must be trivial when using 'one_body_solve()'` and aborts via `libc++abi`
  (an uncaught C++ exception, not a Python traceback, so it can't be caught). Use `U=1e-6` instead:
  it keeps the ED path and leaves the one-body physics untouched.
- **Irrep sectors with unspecified `N` are only legal while something anomalous is switched on.**
  A production sector list like `R0:S0/R1:S0/R2:S0/R3:S0` (see the irrep-sector entry above) leaves
  particle number free, which is correct for a superconducting run. Switch the anomalous bath and
  pairing field off (`db=0`, `D=0`) for a normal-state or one-body check and pyqcm raises
  `sector string R0:S0 defines a non conserved particle number, but particle number is conserved in
  the model`. Pin `N` for the test (`N12:S0`) and keep the free-`N` list for production; don't
  "fix" the production list.
- **`set_parameters()` may only be called once per model** (`ValueError: The function
  set_parameters() can only be called once`). To vary something afterwards use `set_parameter()`
  (singular) per parameter. This bites when a script wants to reset to a clean baseline between
  successive tests in one process.

## kz Brillouin period follows the physical basis, not 1

What: For a model with a non-cubic `set_basis` (e.g. a c-axis vector of length `C/(NZ*A)` in units of
the in-plane lattice constant), the reciprocal-space period along that axis in the reduced k used by
`dispersion()` is **not** 1. Sampling `kz` over `[0, 1)` silently samples a fraction (or several
copies) of the true Brillouin zone.
Why it happens: this is the concrete consequence of "`set_basis` enters the k-phase" in the W90 entry
above. `dispersion()` takes k in units of `2*pi` in the *physical* basis, so a basis vector of length
`L` has period `1/L`.
What to do instead: compute the period explicitly (`period = 1/L`, e.g. `NZ*A/C`) and verify
numerically that `E(kz=0)` and `E(kz=period)` agree to machine precision before trusting any c-axis
dispersion plot or any claim about bilayer/interlayer physics.

## Tied (dependent) parameters must stay out of the `varia` list

What: pyqcm's parameter string accepts dependent parameters written as `X = c*Y` (e.g.
`eb1_3 = 1*eb1_1`). This is the clean way to impose a relation the model should obey but that the
target sectors and the cluster point group do not already enforce — tying two clusters, sites or bath
orbitals that symmetry makes equivalent (`1*`), imposing a relative sign or parity such as a pairing
phase (`-1*`), or locking a ratio fixed by the model definition. The tie tracks the master through the
whole optimization, not just at seed time.
Why it happens: a dependent parameter is derived, so listing it in `varia` as well hands the optimizer
a degree of freedom that the constraint immediately overwrites.
What to do instead: put only the master names in `varia`. Build the varia list and the tie string from
the same code path so they cannot drift apart.

**The general trap behind this: a symmetric seed is self-confirming.** Any symmetry imposed by
*seeding* rather than by *declaring* will appear to hold no matter whether it actually does, because
the symmetric subspace is an invariant manifold of the map being iterated. If the seed respects a
symmetry that the lattice model and cluster also respect, the impurity solution respects it, so the
updated parameters respect it, forever. The gradient out of the manifold is exactly zero. This is not
specific to bath ties, or to CDMFT:

- Untying a tie and re-running from the symmetric converged solution reproduces the tied answer to
  machine precision and reads as independent confirmation that the tie was justified. It is a
  tautology, not evidence.
- A symmetry-broken order parameter seeded at exactly zero stays exactly zero and the run reports "no
  order" whether or not the ordered phase is actually the ground state — the usual way an AFM or SC
  solution gets missed.
- In VCA the same thing appears as a stationarity artifact rather than a fixed point: an order
  parameter at zero is *always* a stationary point of the Potthoff functional by symmetry, so a
  Newton search started there sits still regardless of whether a nontrivial saddle exists nearby.

Round-off can in principle knock a run off the manifold, but slowly and unreliably — never rely on it,
and never treat a run that happened to drift off as the normal case.
What to do instead: perturb *off* the manifold deliberately. Jitter the freed parameters by a few
percent, or start from a deliberately asymmetric converged solution and check the symmetry is
*restored* rather than merely preserved. Only a run that could have left and didn't is evidence the
symmetry holds.

## Converged bath parameters are gauge-dependent

What: **Bath parameters are not observables.** The bath parametrization is many-to-one onto physical
solutions, so two converged runs can print visibly different `eb`/`tb`/`db` tables and still be the
same solution. Read as physics, a gauge move looks like a discovery: an orbital that "decoupled", a
hybridization that "changed sign", a level that "crossed" between two points of a sweep. Nothing in
the CDMFT distance function prefers one gauge branch over another, so a warm start, a reordered bath,
or a jittered seed can silently move you between them.
Why it happens: any transformation of the bath operators that leaves the hybridization function
`Gamma(z)` invariant is a redundancy of the parametrization. Which ones exist depends only on which
channels the bath declares, not on the model or material:

| Redundancy | Acts on | Sends | Present when |
|---|---|---|---|
| Phase / sign | one orbital, `c_b -> -c_b` | `(eb, tb, db) -> (eb, -tb, -db)` | always |
| Permutation | two orbitals in the same symmetry class | swaps their whole `(eb, tb, db)` triple | whenever ≥2 equivalent orbitals |
| Particle-hole flip | one orbital, `c_b -> c_b^dag` | `(eb, tb, db) -> (-eb, db, tb)` | only when normal **and** anomalous channels both exist |

The first two are always live. The sign one means **the sign of an individual `tb_i` carries no
information** — only relative signs within an orbital do. The permutation one is why bath orbitals can
appear to trade places discontinuously along a continuation sweep.

The particle-hole flip is the one that most often gets misread, because it moves weight *between*
channels: it turns the normal hybridization `tb c_i^dag c_b` into an anomalous one and vice versa. So a
converged orbital showing `tb_i` near zero with `db_i` appreciable is usually **not** a decoupled
orbital carrying optimizer noise in its anomalous amplitude — it is the same orbital in the flipped
branch, physically identical to one with `tb_i` appreciable and `db_i` near zero. Every bath orbital
carries its own copy of this Z2. It is exact only because the standard `eb`/`tb`/`db` parametrization
has no bath-internal pairing term; add one and the degeneracy breaks. (This case needs an anomalous
channel to exist, so in practice it shows up in superconducting runs — but it is a property of the
parametrization, not of superconductivity, and the same reasoning applies to any parametrization whose
channels mix under a transformation that preserves `Gamma(z)`.)

What to do instead: before treating a surprising bath table as physics worth pinning away or reporting,
test the gauge hypothesis. It is cheap and decisive, and the recipe is the same for all three rows:

- **Do the parameters map onto each other?** For the flip: does `eb_i` have the opposite sign to the
  same orbital in the neighbouring or previous solution, and does `|db_i|` in one run equal `|tb_i|` in
  the other to optimizer tolerance? For a permutation: does the *multiset* of triples match even though
  the per-index table doesn't?
- **Do the observables match?** `E_kin`, the densities, the order parameters and `min_dist` are
  invariant under all three. This is the check that settles it — gauge moves cannot move an observable.
- **Does `E0` move by the predicted amount?** Sign and permutation leave `E0` alone. The flip shifts it
  by `2 * sum(eb_i)` over the flipped orbitals, taking `eb_i` from the run you are transforming *from*
  (so the shift correctly reverses sign when you flip back), out of `n_b -> 1 - n_b` for both spins.

If those hold, there is nothing wrong and nothing to fix. Genuine noise in a truly decoupled orbital
looks different: `eb` does not flip, and no relation ties the two runs' numbers together.

Two corollaries. First, do not compare `E0` across runs without checking the bath gauge first, or a
pure relabelling reads as a level crossing. Second, pinning `db_i = 0` on a flipped-looking orbital is
not a no-op even though it usually reproduces the same answer: it forbids the small residual pairing
left over after the rotation, which restricts the variational space slightly and can cost a nearby
solution branch in a continuation sweep. If a pinned sweep loses a solution the free sweep had, suspect
that before concluding the free sweep's solution was spurious.
