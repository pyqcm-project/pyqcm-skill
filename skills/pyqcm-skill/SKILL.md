---
name: pyqcm-skill
description: Assists with pyqcm, the Python/C++ library for quantum cluster methods (CPT, VCA, CDMFT) used to study strongly correlated electron models like the Hubbard model. Use this whenever the user is writing or debugging a pyqcm script (defining clusters, lattice models, operators, parameters, running model_instance calculations), interpreting pyqcm output (spectral functions, Green's functions, self-energy, order parameters, ground state averages, phase diagrams), installing or building pyqcm, or reading/modifying the pyqcm source itself (the src_ed exact-diagonalization solver, the src_qcm lattice/CPT-VCA-CDMFT engine, or the src_python nanobind bindings). Also trigger on mentions of cluster perturbation theory, exact diagonalization impurity solver, Lanczos, Lehmann representation, or the qcm_wed/pyqcm-project repository, even if the user doesn't say "pyqcm" by name.
---

# pyqcm

Pyqcm implements three related quantum cluster methods, Cluster Perturbation Theory (CPT), the
Variational Cluster Approach (VCA), and Cellular/Cluster Dynamical Mean Field Theory (CDMFT), for
approximating strongly correlated (Hubbard-like) lattice models. The impurity solver is exact
diagonalization (Lanczos and variants) on sparse matrices. Core numerics are C++; the day-to-day
interface is Python.

## First: locate the pyqcm source tree

This skill ships no pyqcm source. Nearly everything below refers to a **pyqcm checkout**, written
`$PYQCM_ROOT`. Resolve it before doing anything else, in this order:

1. `$PYQCM_ROOT`, if the user has it set.
2. Derive it from an installed package:
   `python3 -c "import pyqcm, pathlib; print(pathlib.Path(pyqcm.__file__).resolve().parent.parent)"`,
   then **confirm `src_ed/` exists there**. A pip-installed pyqcm ships only the Python package plus
   the compiled extension, so this check fails for wheel installs and that is the expected outcome,
   not an error.
3. Ask the user for the path.

If no checkout is reachable, say so plainly and carry on: the scripting and physics jobs still work
from the reference files alone. Do not hand out paths under `$PYQCM_ROOT` that you have not confirmed
exist.

The checkout is also the install source. pyqcm on PyPI is **sdist-only**, so every install compiles
from source and needs CMake, a C++ compiler, and BLAS regardless of how it is invoked.

## The four jobs

Work out which one the user needs (often more than one at once, for example debugging a script *and*
interpreting why the physics looks wrong) and read the matching reference before acting:

| Job | When | Read |
|---|---|---|
| Set up or fix an installation | `import pyqcm` fails, "unable to load the QCM library", a build error, a fresh venv, moving to a cluster | `references/install.md`, plus `$PYQCM_ROOT/INSTALL.md` for the full `CMAKE_ARGS` catalogue |
| Write or debug a script | Defining clusters/models/operators, running ED/CDMFT/VCA, fixing a traceback, choosing sectors | `references/scripting.md` |
| Interpret physics results | Explaining a spectral function, self-energy, order parameter, or phase diagram; connecting output to the underlying theory | `references/physics.md` |
| Modify pyqcm itself | Touching `$PYQCM_ROOT/src_ed/`, `src_qcm/`, `src_python/`, or the pure-Python `pyqcm/*.py` wrapper | `references/modifying-pyqcm.md` |

**Always check `references/practice.md` first**, whichever job it is. It is the single running record
of pyqcm's sharp edges and of how this group actually works, hand-written by the maintainer and
written down nowhere else. Skim its headings for the topic at hand rather than trusting any summary
of its contents.

**Also read `references/physics.md`'s "Grounding claims in the literature" section whenever the work
targets a specific named material or compound**, rather than a generic toy Hubbard model, even when
the job otherwise classifies as pure scripting. The table above routes to one reference per job, which
is not enough here. `references/practice.md`, "Grounding a named material in the literature", says
why.

## Other references

- `references/bath-parametrizer.md`: symmetry-constrained CDMFT bath parametrization via the separate
  `bath-parametrizer` package. `get_pyqcm_generators()` is the one-call path to
  `cluster_model(generators=..., bath_irrep=True)`, with or without subbaths.
- `references/hpc.md`: running pyqcm in a module-based HPC environment, using the Institut quantique
  cluster as a worked example. Read it before advising on any cluster install or SLURM job.
- `references/research/CITATIONS.md`: the papers this skill cites, by arXiv id and DOI, grouped into
  quantum cluster methods, periodization, cuprates, and double counting. Ground physics explanations
  in these papers, not just intuition. **Paper full texts are not stored in this repo**; the manifest
  has a fetch command, and fetched copies are git-ignored.

## Coding conventions (always follow these)

Always consult `references/guidelines.md`. They apply whenever this skill is used to write or modify
code, scripts, prose, or commits.

## Checking build state before debugging

The compiled `qcm` extension must be built before any script can run. Pure Python imports of `pyqcm`
succeed even when the extension is missing, but simulations fail. Check with:

```bash
python3 -c "import pyqcm" 2>&1 | tail -3
```

If it reports it "was unable to load the QCM library", it needs building. Go to `references/install.md`
before assuming a code change is broken. Do not chase a phantom bug in Python logic when the real
issue is a stale or missing build.

## pyqcm layout, relative to `$PYQCM_ROOT`

- `pyqcm/`: the Python package (`__init__.py` is the main API surface: `cluster_model`, `cluster`,
  `lattice_model`, `model_instance`, etc.). A few submodules are pure Python (`cdmft.py`, `vca.py`,
  `_loop.py`, `_spectral.py`, `_draw.py`, `green_structure.py`). This is the only part a pip install
  puts in your venv.
- `src_ed/`: the C++ exact-diagonalization impurity solver (Lanczos, Green's functions, sectors,
  symmetry).
- `src_qcm/`: the C++ lattice/CPT-VCA-CDMFT engine (periodization, Green's functions on the lattice,
  parameter sets).
- `src_python/`: nanobind bindings gluing the C++ core into the `pyqcm.qcm` extension module.
- `docs/source/*.rst`: the authoritative API/workflow documentation. Build it locally (see
  `references/install.md`) for rendered HTML, or read
  https://qcm-wed.readthedocs.io/.
- `notebooks/*.py` / `*.ipynb`: worked examples (1D Hubbard chains, antiferromagnetism,
  superconductivity, CDW, Rashba coupling, graphene Mott transition), the best source of idiomatic
  usage patterns.
- `tests/`: `test_all.py` runs everything; individual tests live in `tests/test_files/`.
- `INSTALL.md`: upstream's install instructions and the full build-option catalogue.

## Contributing a gotcha back

This skill is public and takes pull requests. At the **end** of a session, suggest a contribution to
`references/practice.md` only when all of these hold:

- **General**: the lesson would have changed the outcome for someone who is not this user, on a
  different model or system.
- **Durable**: it is a property of pyqcm or of the method, not of one script, one dataset, or one
  parameter choice.
- **It cost something real**: wasted time, a wrong result, a discarded run, or a genuinely confusing
  error. Merely interesting is not enough.

When it does fire, encourage the user to write the entry manually. Then, you may format it as excpected
from `CONTRIBUTING.md` so the maintainer's review is a yes or no rather than an editing job.
Never edit `references/practice.md` yourself: it is hand-written and hand-maintained by the author,
and the installed copy is overwritten on plugin update anyway, so the change would be silently lost.
Propose it to them instead. See `CONTRIBUTING.md` at the repo root.
