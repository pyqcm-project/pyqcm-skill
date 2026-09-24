# Modifying pyqcm itself

pyqcm has a pure-Python layer on top of four C++ ones. Know which layer a change belongs in
before writing code: the same concept (e.g. "an operator") exists at multiple layers with different
responsibilities, and touching the wrong one either won't compile or won't be reachable from Python.

## Layers, outside-in

1. **`$PYQCM_ROOT/pyqcm/*.py`**: pure Python. This is where the user-facing API lives (`__init__.py`, about
   3300 lines) plus higher-level pure-Python logic that doesn't need C++ performance: `cdmft.py`
   (CDMFT self-consistency driver), `vca.py` (VCA driver), `_loop.py` (parameter sweeps),
   `_spectral.py`, `_draw.py`, `green_structure.py` (Green's function representations: Lehmann,
   continued fraction, rational fraction, moments), `slab.py`, `variable_parameter.py`. If a change
   is about *orchestrating* calculations (a new sweep strategy, a new self-consistency variant, a new
   way to post-process a Green's function) rather than about the ED solver or lattice engine
   themselves, it likely belongs here and can be developed and tested without touching CMake at all.

2. **`$PYQCM_ROOT/src_python/`**: the nanobind glue (`qcm_lib.cpp` is the module entry point,
   `qcm_wrap.hpp`/`qcm_ED_wrap.hpp` register the actual bound functions, `common_Py.cpp` has shared
   conversion helpers). New C++ functionality only becomes callable from Python once it's registered
   here. If a new C++ function exists but `pyqcm.<name>` raises `AttributeError`, check whether it
   was actually exposed through this layer.

3. **`$PYQCM_ROOT/src_qcm/`**: the lattice/CPT-VCA-CDMFT engine: `lattice_model`/`lattice_model_instance`
   (the lattice-level model and its solved instances), `lattice_operator` (lattice Hamiltonian
   terms), `CPT.cpp` (periodization), `Chern.cpp` (topological invariants), `parameter_set.cpp`
   (lattice parameter bookkeeping), `Green_function.hpp`, `basis3D.cpp`/`lattice3D.cpp` (geometry).
   `QCM.hpp`/`QCM.cpp` is the public C++ interface (free functions in the `QCM` namespace) that the
   Python bindings call into, a reasonable first stop when tracing "what does calling X from Python
   actually do." `CPT.cpp` is specifically where the existing G/M/S/C/N periodization schemes are
   computed, and the natural landing spot if and when Liouvillian interpolation (see
   `references/physics.md`, "Periodization / interpolation schemes") gets implemented as a new
   scheme, though as of this writing that work hasn't started. Confirm current status before assuming
   otherwise.

4. **`$PYQCM_ROOT/src_ed/`**: the exact-diagonalization impurity solver, independent of any lattice
   concept. `model.hpp`/`model.cpp` define a cluster's parameter-independent structure;
   `model_instance*.cpp` solve it for specific parameter values; `sector.cpp`/`ED_basis.cpp` handle
   Hilbert space sectors and symmetry-adapted bases; `Hamiltonian/` holds the different Hamiltonian
   storage/solve strategies (CSR sparse, Eigen, on-the-fly, factorized) and the Lanczos/Davidson/PRIMME
   eigensolvers; `Operators/` holds each Hermitian operator type (one-body, interaction, anomalous,
   Hund, Heisenberg, general interaction) both in second-quantized form and in its Hilbert-space
   (`HS_*`) matrix-element form; `continued_fraction*.cpp`/`matrix_continued_fraction.cpp` compute
   the cluster Green's function from the solved ground state via continued fractions.

5. **`$PYQCM_ROOT/src_util/`**: shared numerical utilities not specific to ED or lattice logic: dense and
   sparse matrix wrappers (`matrix.cpp`), BLAS/LAPACK shims (`lapack-blas.h`, `cblas.h`), HDF5 I/O
   (`hdf5_io.cpp`), numerical integration (`integrate.cpp`), the input parser (`parser.cpp`).

A new *operator type* (e.g. a new kind of interaction term) touches layer 4 (define it, likely
alongside its `HS_*` counterpart) and probably layer 3 (if it needs lattice-level periodization
handling) and layer 2 (if new parameters need exposing to Python). Check `models.rst` first to see
if something close already exists to model the new operator on, since these operator classes follow
a consistent pattern.

## Build/test loop while developing

- Rebuild after any C++ change: see `references/practice.md` (editable installs still trigger a full
  CMake reconfigure and rebuild; there's no standalone incremental-only path via `pip install -e .`).
- Run `$PYQCM_ROOT/tests/test_all.py` after non-trivial changes to `src_ed`/`src_qcm`. A change meant to be
  purely additive can silently perturb existing solvers, e.g. via shared Hamiltonian-construction
  code paths.
- If a change only touches `$PYQCM_ROOT/pyqcm/*.py`, no C++ rebuild is needed at all; just re-run the
  affected script or test.
- `$PYQCM_ROOT/bench/` (ED, integrals, VCA) holds performance benchmarks. Check these if a change could
  plausibly affect performance-sensitive code paths (Hamiltonian construction and diagonalization,
  integration routines).

## Style

`src_ed/.clang-format` defines the C++ formatting convention for that directory. Run it, or match its
style by eye, rather than introducing a different style within the same files.
