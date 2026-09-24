# bath-parametrizer: symmetry-constrained CDMFT baths

`bath-parametrizer` derives pyqcm `cluster_model` generators from a point group, so you do not hand-
derive the `generators=[...]` / `bath_irrep=True` encoding for a symmetric cluster. Deriving it by
hand is error-prone: each generator row encodes both the cluster-site permutation and the phase every
bath orbital picks up under that group operation.

It is a separate package, not part of pyqcm and not bundled with this skill. Source:
https://github.com/antoinedelagrave/pyqcm-bath-parametrizer

## Setup

```bash
git clone https://github.com/antoinedelagrave/pyqcm-bath-parametrizer.git
cd pyqcm-bath-parametrizer
pip install -e .
```

Editable install, so edits to the checkout take effect immediately. It is pure Python and needs no
compilation, unlike pyqcm itself.

Import path is `bath_parametrizer.bath_parametrization`:

```python
from bath_parametrizer.bath_parametrization import BathParametrizer
```

The package README shows a `src/bath_parametrization/` layout and a `from bath_parametrization
import ...` line that do not match the shipped package. Trust the import above.

## The one call that matters

`get_pyqcm_generators()` goes straight to a pyqcm cluster model:

```python
sites = [[0, 0, 0], [1, 0, 0], [0, 1, 0], [1, 1, 0]]   # list or ndarray both work
p = BathParametrizer(sites, "C2v")

generators = p.get_pyqcm_generators(8, "C2v")          # 8 bath orbitals, 2 per C2v irrep
pyqcm.cluster_model(4, 8, generators=generators, bath_irrep=True)
```

```
get_pyqcm_generators(n_baths, abelian_pg, subbath=None, linked_sites=None) -> list | dict
```

- `n_baths`: total bath orbitals.
- `abelian_pg`: point group supplying the generators.
- `subbath`: `{"nsb": <count>, "irreps": "replica" | "unique"}`. Default is one subbath, `"replica"`,
  which is the vanilla non-SB-CDMFT mixed bath. `"custom"` and `"mixed"` raise `NotImplementedError`.
- `linked_sites`: 0-based cluster sites physically coupled to the bath, defaulting to all of them.
  Must be a union of point-group orbits or it raises `ValueError`.

Returns a flat generators list when `nsb` is 1, and a dict keyed by 1-based subbath index when
`nsb > 1`, one flat list per subbath cluster model. Check which you got before passing it on.

Bath-phase blocks follow the SALC label order, so declare your `eb{i}`/`tb{i}` in that same order. A
mismatch silently couples the wrong bath orbitals, which is the nastiest failure mode here since
nothing complains.

For SB-CDMFT:

```python
gens = p.get_pyqcm_generators(8, "C2v", subbath={"nsb": 3, "irreps": "replica"})
# {1: [...], 2: [...], 3: [...]}
```

## The rest of the API

- `get_bath_parametrization() -> dict`: the SALCs per irrep, `{irrep: [salc, ...]}`. Use it to see
  what the symmetry analysis produced before committing to a bath size.
- `get_hybridization_links(nb, subbath=None, linked_sites=None) -> dict`: assigns bath orbitals to
  SALCs per subbath, returning
  `{subbath_index: {salc_label: {"coefficients": ndarray, "n_orbitals": int}}}`. `get_pyqcm_generators`
  delegates to it, so both take the same `subbath`/`linked_sites` arguments. Call it directly when you
  need the orbital counts per SALC, which is what tells you how many `eb{i}`/`tb{i}` parameters to
  declare and in what order.
- `show_parametrized_cluster(show=True) -> Axes`: plots the cluster with its symmetry labels. Use it
  to confirm the geometry and point group you passed actually describe the cluster you meant.
- `project_onto_irrep(...)`, `build_permutation_repr(operation)`: the lower-level machinery, if you are
  checking the symmetry analysis rather than using it.

## Supported point groups

`Cs`, `C2`, `C2v`, `C3`, `C3v`, `C4`, `C4v`, `C6`, `C6v`. All abelian. `bath_parametrizer.point_groups`
exports `all_point_groups`.

An unsupported or non-abelian group yields an empty dict from `get_pyqcm_generators`, which is the
signal that `bath_irrep` symmetry is unavailable for that cluster. Fall back to `bath_irrep=False`.
