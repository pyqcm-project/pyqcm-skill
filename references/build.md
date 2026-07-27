# Building pyqcm

pyqcm's Python layer (`pyqcm/pyqcm/*.py`) is pure Python and always importable. The actual physics
lives in a compiled extension module (`pyqcm.qcm`) built from `src_ed/`, `src_qcm/`, and
`src_python/` via CMake + scikit-build-core + nanobind (see `pyqcm/pyproject.toml` and
`pyqcm/CMakeLists.txt`). If that extension isn't built, `import pyqcm` still succeeds but prints:

```
pyqcm was unable to load the QCM library. You will not be able to run simulations...
Please reinstall pyqcm!
```

Diagnose *that* before debugging what looks like a Python bug — a stale or missing build produces
confusing symptoms (e.g. `AttributeError` on things that clearly exist in `__init__.py`, or silent
`qcm = None`).

## Prerequisites

- CMake (`brew install cmake` on macOS, `apt install cmake` on Debian/Ubuntu)
- A BLAS/LAPACK implementation — on macOS the Accelerate framework is picked up automatically; on
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
source changes — there's no incremental C++ rebuild wired into `pip install -e .` on its own, so
after touching anything under `src_ed/`, `src_qcm/`, or `src_python/`, re-run the install command
to pick up the change. If iterating quickly on C++, consider driving CMake directly instead
(configure once into a build dir, then `cmake --build`) to get incremental compilation — see
`pyqcm/CMakeLists.txt` for the target layout.

To customize the build (BLAS vendor, PRIMME eigensolver, Eigen Hamiltonian), set `CMAKE_ARGS` before
installing, e.g.:

```bash
export CMAKE_ARGS="-DBLA_VENDOR=OpenBLAS -DWITH_PRIMME=1 -DDOWNLOAD_PRIMME=1"
uv pip install -e . --no-build-isolation
```

Full option descriptions are in `pyqcm/INSTALL.md` and `pyqcm/docs/source/intro.rst`.

## Verifying the build

```bash
python3 -c "import pyqcm; print(pyqcm.__version__)"
```

should print a version with no "unable to load" warning. Then a quick smoke test against a known
example:

```bash
cd pyqcm && python3 notebooks/intro1.py
```

For a fuller check, `pyqcm/tests/test_all.py` runs the whole test suite (outputs land in
`tests/test_outputs/`); individual tests under `tests/test_files/` can be run standalone when you
only need to check one solver path.

## Building the docs

```bash
cd pyqcm/docs && ./makedoc
```

produces HTML under `docs/html/`. There's no pre-built copy checked into this repo (the docs are
fully contained in the `pyqcm` submodule itself, so mirroring a build here was redundant) — rebuild
locally when you need to browse them.
