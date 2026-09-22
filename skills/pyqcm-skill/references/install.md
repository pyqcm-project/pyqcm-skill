# Installing and building pyqcm

pyqcm's Python layer (`$PYQCM_ROOT/pyqcm/*.py`) is pure Python and always importable. The actual physics
lives in a compiled extension module (`pyqcm.qcm`) built from `src_ed/`, `src_qcm/`, and
`src_python/` via CMake + scikit-build-core + nanobind (see `$PYQCM_ROOT/pyproject.toml` and
`$PYQCM_ROOT/CMakeLists.txt`). If that extension isn't built, `import pyqcm` still succeeds but prints:

```
pyqcm was unable to load the QCM library. You will not be able to run simulations...
Please reinstall pyqcm!
```

Diagnose *that* before debugging what looks like a Python bug. A stale or missing build produces
confusing symptoms, e.g. `AttributeError` on things that clearly exist in `__init__.py`, or silent
`qcm = None`.

## Getting a working install

pyqcm on PyPI is **source-only** (sdist, no wheels), so `pip install pyqcm` compiles from source just
like a checkout does. There is no binary fast path. Every install needs CMake, a C++ compiler, and
BLAS, so work through Prerequisites below first.

The canonical flow, which is also what upstream `$PYQCM_ROOT/INSTALL.md` prescribes, is to install
from a checkout inside a virtual environment:

```bash
git clone https://github.com/pyqcm-project/pyqcm.git
cd pyqcm
python3 -m venv .venv && source .venv/bin/activate   # or uv venv, or conda
pip install .
```

Use `pip install -e .` instead if you will be editing pyqcm itself.

Before installing, decide whether you want the non-default build options (PRIMME, Eigen, a specific
BLAS vendor). They change performance meaningfully and are awkward to revisit later, so surface them
to the user rather than silently taking defaults. See "Building" below and
`$PYQCM_ROOT/INSTALL.md` for the full catalogue.

**On an HPC cluster, read `references/hpc.md` first.** Module-based environments need a specific
`module load` sequence and specific `CMAKE_ARGS`, and getting the BLAS choice wrong there causes a
severe, easily misdiagnosed slowdown.

## Prerequisites

- CMake (`brew install cmake` on macOS, `apt install cmake` on Debian/Ubuntu)
- A BLAS/LAPACK implementation. On macOS the Accelerate framework is picked up automatically; on
  Linux install `libopenblas-dev` (or similar) or point at one with `-DBLA_VENDOR=...`
- Eigen (`libeigen3-dev` on Debian/Ubuntu) if building with `EIGEN_HAMILTONIAN=1` (the default)
- A C/C++ compiler. On Apple platforms, CMakeLists.txt forces `clang`/`clang++` unless you override
  `CMAKE_C_COMPILER`/`CMAKE_CXX_COMPILER` explicitly.

## Building

From the `pyqcm/` directory (the repo root containing `pyproject.toml`):

```bash
uv pip install -e . --no-build-isolation
```

or, without `uv`:

```bash
pip install -e .
```

Editable installs still trigger a full CMake/scikit-build-core rebuild whenever the extension's
source changes; there's no incremental C++ rebuild wired into `pip install -e .` on its own, so
after touching anything under `src_ed/`, `src_qcm/`, or `src_python/`, re-run the install command
to pick up the change. If iterating quickly on C++, consider driving CMake directly instead
(configure once into a build dir, then `cmake --build`) to get incremental compilation. See
`$PYQCM_ROOT/CMakeLists.txt` for the target layout.

To customize the build (BLAS vendor, PRIMME eigensolver, Eigen Hamiltonian), set `CMAKE_ARGS` before
installing, e.g.:

```bash
export CMAKE_ARGS="-DBLA_VENDOR=OpenBLAS -DWITH_PRIMME=1 -DDOWNLOAD_PRIMME=1"
uv pip install -e . --no-build-isolation
```

Full option descriptions are in `$PYQCM_ROOT/INSTALL.md` and `$PYQCM_ROOT/docs/source/intro.rst`.

## Verifying the build

```bash
python3 -c "import pyqcm; print(pyqcm.__version__)"
```

should print a version with no "unable to load" warning. Then a quick smoke test against a known
example:

```bash
cd $PYQCM_ROOT && python3 notebooks/intro1.py
```

For a fuller check, `$PYQCM_ROOT/tests/test_all.py` runs the whole test suite (outputs land in
`tests/test_outputs/`); individual tests under `tests/test_files/` can be run standalone when you
only need to check one solver path.

## Building the docs

```bash
cd $PYQCM_ROOT/docs && ./makedoc
```

produces HTML under `docs/html/`. This skill ships no pre-built copy, so rebuild locally when you
need to browse them, or read the hosted build at https://qcm-wed.readthedocs.io/.
