# Writing and debugging pyqcm scripts

Every pyqcm calculation follows the same shape, whether the underlying method ends up being plain
CPT, VCA, or CDMFT. `pyqcm/notebooks/intro1.py` is the canonical minimal example; read it before
writing a new script, then adapt from the closest matching notebook in `pyqcm/notebooks/` rather than
starting from a blank page. There is already a worked example for AF order, CDW, superconductivity,
Rashba coupling, slab/heterostructure geometries, and graphene.

`pyqcm/tests/test_files/` is a second, larger source of worked examples. It is the actual test suite
(auto-discovered and run by `test_all.py`), so every file there is a real, currently-passing script,
not illustrative pseudocode. It covers things `notebooks/` doesn't: subbath CDMFT
(`test_cdmft_sb.py`, `test_cdmft_1x4_4b_2sb.py`), multi-cluster models (`test_2clusters.py`,
`test_cdmft_2clusters.py`), Hartree-mean-field treatment (`test_cdmft_hartree.py`), different CDMFT
minimization methods (`test_cdmft_methods.py`, `test_cdmft_jac_methods.py`, `test_cdmft_trf.py`), and
periodization (`test_periodize.py`). When a notebook doesn't cover the feature in question, check
here before assuming it isn't demonstrated anywhere in the codebase.

## The standard workflow

1. **Cluster model**: `pyqcm.cluster_model(n_sites, ...)` declares the shape of one impurity
   (physical sites, and optionally bath sites for CDMFT).
2. **Cluster**: `pyqcm.cluster(cluster_model, positions)` places a physical instance of that model
   at specific lattice sites. Passing a **list** of `cluster_model` objects instead of one here (each
   with its own bath) is how *subbath CDMFT* is set up, not a separate API. See
   `references/physics.md` for what subbath CDMFT buys physically and why this group wants it used,
   and `pyqcm/docs/source/cdmft.rst` ("Subbaths") for the mechanics (self-energies and
   hybridizations from each system are averaged by default when building the host).
3. **Lattice model**: `pyqcm.lattice_model(name, clusters, superlattice_vectors)` tiles the
   cluster(s) into the infinite lattice.
4. **Operators**: declare the Hamiltonian terms on the model: `interaction_operator` (Hubbard U),
   `hopping_operator` (one-body/hopping), plus anomalous, density-wave, Hund, and Heisenberg
   operators for more exotic order. See `pyqcm/docs/source/models.rst` for the full operator taxonomy
   and the sign/normalization conventions; get this wrong and the physics is subtly wrong without
   erroring.
5. **Target sectors**: `model.set_target_sectors('R0:N4:S0')` restricts the Hilbert space search
   (irrep : particle number : spin). If the ground state search fails or lands somewhere physically
   implausible, the sector string is one of the first things to check.
6. **Parameters**: `model.set_parameters(...)` sets numeric values for every declared operator.
   Every operator declared in step 4 needs a value here, and vice versa; a mismatch is a common
   source of errors.
7. **Instance**: `pyqcm.model_instance(model)` builds a concrete solvable instance from the current
   parameter values. For VCA/CDMFT you don't usually call this directly, since `pyqcm.vca` and
   `pyqcm.cdmft` drive a sequence of instances while varying variational/bath parameters.
8. **Solve / query**: `I.ground_state()`, `I.cluster_averages()`, `I.cluster_spectral_function()`,
   `I.spectral_function()`, `I.plot_DoS()`, etc.

## Where to look for each piece

| Topic | Doc source |
|---|---|
| Cluster model definition, bath sites | `pyqcm/docs/source/defining_cluster_models.rst` |
| Lattice model definition, superlattice vectors | `pyqcm/docs/source/defining_models.rst` |
| Operator types (hopping, anomalous, density wave, Hund, Heisenberg), mixing states, orbital/sector index conventions | `pyqcm/docs/source/models.rst` |
| Parameter sets, `set_parameters` syntax | `pyqcm/docs/source/parameters.rst` |
| CDMFT self-consistency loop, incl. subbath (multiple bath systems per cluster) | `pyqcm/docs/source/cdmft.rst` |
| VCA (Potthoff functional minimization) | `pyqcm/docs/source/vca.rst` |
| Spectral functions, Green's functions, periodization | `pyqcm/docs/source/spectral.rst`, `pyqcm/pyqcm/green_structure.py` (docstrings; the rendered `green_structure` doc page is autodoc-generated from this module, not a standalone `.rst`) |
| Hartree/mean-field treatment of extended interactions | `pyqcm/docs/source/hartree.rst` |
| Sweeping a parameter (phase diagrams) | `pyqcm/docs/source/loop.rst` |
| Plotting/drawing clusters and operators | `pyqcm/docs/source/draw.rst` |
| Slab / heterostructure / hybrid lattice geometries | `pyqcm/docs/source/lattice_hybrid.rst` |
| Parallelism (multi-core ED, integrals) | `pyqcm/docs/source/parallel.rst` |
| Everything else (misc functions) | `pyqcm/docs/source/other_functions.rst` |

If searching rendered docs is easier than grepping `.rst` files, build them locally with
`cd pyqcm/docs && ./makedoc` (see `references/build.md`). The output includes the C++ API reference
(`cpp_api/`) which isn't mirrored as `.rst`.

## Debugging checklist

Before diving into the C++ layer, rule out the cheap stuff first:

1. **Is the extension actually built?** See `references/build.md`; a missing or stale build produces
   errors that look like Python bugs.
2. **Do operators and parameters match 1:1?** Every operator declared via `*_operator(...)` needs a
   corresponding entry in `set_parameters(...)`, and vice versa.
3. **Is the target sector consistent with the model?** Wrong particle number or spin sector for the
   declared operators is a frequent cause of a ground state search that fails or silently returns
   nonsense.
4. **Does the mixing state match what you intended?** Adding an anomalous or spin-flip operator
   changes the Green function's mixing state (normal, then spin-asymmetric, spin-flip, Nambu; see
   `models.rst`, "Green function indices and mixing states"), which changes matrix shapes downstream.
   A shape mismatch several steps later often traces back to an operator added earlier than expected.
5. **Compare against a close notebook example** rather than the abstract docs. If a similar model
   already works in `pyqcm/notebooks/`, diff against it.
6. **Check `references/gotchas.md`** for anything already known about this exact failure mode.

## Verifying a fix

Run the specific example script (`python3 notebooks/<name>.py`) to confirm output looks sane before
declaring something fixed, and if the change could plausibly affect other models, run
`pyqcm/tests/test_all.py` (outputs land in `tests/test_outputs/`) rather than assuming a local fix is
isolated.
