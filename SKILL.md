---
name: pyqcm
description: Assists with pyqcm, the Python/C++ library for quantum cluster methods (CPT, VCA, CDMFT) used to study strongly correlated electron models like the Hubbard model. Use this whenever the user is writing or debugging a pyqcm script (defining clusters, lattice models, operators, parameters, running model_instance calculations), interpreting pyqcm output (spectral functions, Green's functions, self-energy, order parameters, ground state averages, phase diagrams), or reading/modifying the pyqcm source itself (the src_ed exact-diagonalization solver, the src_qcm lattice/CPT-VCA-CDMFT engine, or the src_python nanobind bindings). Also trigger on mentions of cluster perturbation theory, exact diagonalization impurity solver, Lanczos, Lehmann representation, or the qcm_wed/pyqcm-project repository, even if the user doesn't say "pyqcm" by name.
---

# pyqcm

Pyqcm implements three related quantum cluster methods — Cluster Perturbation Theory (CPT), the
Variational Cluster Approach (VCA), and Cellular/Cluster Dynamical Mean Field Theory (CDMFT) — for
approximating the physics of strongly correlated (Hubbard-like) lattice models. The impurity solver
is exact diagonalization (Lanczos and variants) on sparse matrices. Core numerics are C++; the
day-to-day interface is Python.

This skill covers three distinct jobs. Figure out which one the user needs (often more than one at
once — e.g. debugging a script *and* interpreting why the physics looks wrong) and read the matching
reference file before acting:

| Job | When | Read |
|---|---|---|
| Write or debug a pyqcm script | Defining clusters/models/operators, running ED/CDMFT/VCA, fixing a traceback, choosing sectors | `references/scripting.md` |
| Interpret physics results | Explaining a spectral function, self-energy, order parameter, or phase diagram; connecting output to the underlying theory | `references/physics.md` |
| Modify pyqcm itself | Touching `src_ed/`, `src_qcm/`, `src_python/`, or the pure-Python `pyqcm/*.py` wrapper | `references/contributing.md` |

**Always check `references/gotchas.md` first**, regardless of which job it is. It's a running list
of hard-won lessons from this specific research group about pyqcm's sharp edges — conventions, past
mistakes, build quirks — that aren't written down anywhere else. If you learn a new one during a
session (a mistake you made, a surprising API behavior, a fix that wasn't obvious from the docs),
add it there before finishing, so the next session benefits too.

**Also check `references/physics.md`'s "Grounding claims in the literature" section whenever the work
targets a specific named material or system** (a real compound, not a generic toy Hubbard model),
even if the job otherwise classifies as pure scripting. Setting model parameters, choosing a
cluster/bath decomposition, or reasoning about expected order-parameter behavior are
physics-grounding decisions regardless of whether the file being edited is a Python script — the job
table above picks *one* reference file per session, which is not enough by itself when a real,
previously-studied material is involved. See "Grounding a named material/system in the literature" in
`references/gotchas.md`.

## Coding conventions (always follow these)

These apply whenever this skill is used to write or modify code, scripts, or commits (not to prose
explanations of physics, where standard Greek notation is correct and expected):

1. **No AI-tell characters in code, comments, or commit messages.** No em dashes, no Greek unicode
   letters. Spell things out (`Delta`, `Sigma`, `mu`) or match whatever ASCII convention the
   surrounding pyqcm code already uses instead.
2. **Do not overcomment.** Comments should be rare, short, and only explain a non-obvious "why" (a
   workaround, a hidden constraint). Never restate what the code already makes clear. Match pyqcm's
   own sparse commenting style.
3. **Do not over-engineer.** If a simple, direct implementation solves the task, use that. Do not add
   abstractions, configurability, or generality the task did not ask for.
4. **Never commit or push.** Staging, writing the commit message, and pushing are the user's job, not
   this skill's, every time, regardless of how the request is phrased or how large the diff is. The
   user is expected to know git/GitHub themselves; do not offer to do it for them.

## Repo layout

- `pyqcm/pyqcm/` — the Python package (`__init__.py` is the main API surface: `cluster_model`,
  `cluster`, `lattice_model`, `model_instance`, etc.). A few submodules are pure Python
  (`cdmft.py`, `vca.py`, `_loop.py`, `_spectral.py`, `_draw.py`, `green_structure.py`).
- `pyqcm/src_ed/` — the C++ exact-diagonalization impurity solver (Lanczos, Green's functions,
  sectors, symmetry).
- `pyqcm/src_qcm/` — the C++ lattice/CPT-VCA-CDMFT engine (periodization, Green's functions on the
  lattice, parameter sets).
- `pyqcm/src_python/` — nanobind bindings gluing the C++ core into the `pyqcm.qcm` extension module.
- `pyqcm/docs/source/*.rst` — the authoritative API/workflow documentation. Build it locally (see
  `references/build.md`) if you need the rendered HTML; there's no pre-built copy in this repo.
- `pyqcm/notebooks/*.py` / `*.ipynb` — worked examples (1D Hubbard chains, antiferromagnetism,
  superconductivity, CDW, Rashba coupling, graphene Mott transition, etc.) — the best source of
  idiomatic usage patterns.
- `pyqcm/tests/` — `test_all.py` runs everything; individual tests live in `tests/test_files/`.
- `references/pyqcm-bath-parametrizer/` — submodule providing symmetry-constrained bath
  parametrization for CDMFT (point-group generators, SALC-based hybridization). See
  `references/gotchas.md` for usage notes.
- `references/research/` — papers on the underlying theory, sorted by topic into
  `quantum_cluster_methods/`, `periodization/`, and `cuprates/` — ground physics explanations in
  these, not just intuition.

## Build state

The compiled `qcm` extension has to be built before any script can actually run (pure Python imports
of `pyqcm` succeed even when the extension is missing, but simulations will fail). Check quickly with:

```bash
python3 -c "import pyqcm" 2>&1 | tail -3
```

If it reports it "was unable to load the QCM library," it needs building — see
`references/build.md` for the build workflow and common failure modes on this machine before assuming
a code change is broken. Don't chase a phantom bug in Python logic when the real issue is a stale or
missing build.
