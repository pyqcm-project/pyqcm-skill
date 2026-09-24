# Interpreting pyqcm physics results

Don't treat pyqcm output as opaque numbers to report back. Connect it to what the method is actually
approximating: get the mapping between *output quantity* and *physical meaning* right.

**Report, then route.** Say what a quantity *is* and what the method can and cannot support as a
conclusion. Do not supply the physics interpretation itself from general intuition about Hubbard
models: the specifics of CPT/VCA/CDMFT matter, and this group's models are not generic single-band
ones. When the work targets a specific material or a named approximation, point at the relevant paper
in "Related reading" below and hand the interpretation back to the user.

## Which method is being used, and what that implies

- **CPT** (Cluster Perturbation Theory): exact-diagonalizes isolated clusters, then reconnects them
  perturbatively (strong-coupling perturbation theory in the inter-cluster hopping). No
  self-consistency loop. Good for spectral properties (ARPES-like spectral function, DOS) at fixed
  parameters; cannot capture symmetry-broken order on its own.
- **VCA** (Variational Cluster Approach): adds fictitious symmetry-breaking (or bath) terms to the
  cluster Hamiltonian as variational parameters, and extremizes the Potthoff self-energy functional
  over them. A stationary point (not necessarily a minimum in every direction, so check which
  parameters are saddle-point vs. minimum) signals the true value of that variational parameter; its
  magnitude away from zero is the order parameter. See `$PYQCM_ROOT/docs/source/vca.rst`.
- **CDMFT** (Cellular/Cluster DMFT): clusters are coupled to a bath of uncorrelated orbitals whose
  parameters are tuned so the cluster's hybridization function best reproduces the lattice
  self-consistency condition, iterated to convergence. This captures local dynamical correlations
  (unlike VCA/CPT) at the cost of the bath-fitting approximation. See `$PYQCM_ROOT/docs/source/cdmft.rst`.
- **Subbath CDMFT**: a variant published by this group, where a cluster is associated with *more than
  one* independent bath system instead of one. In pyqcm this is not a special mode or class: it is
  implemented by passing a **list** of `cluster_model` objects (instead of a single one) to
  `pyqcm.cluster()`, each with its own bath. Each such "system" gets its own self-energy Sigma_i and
  hybridization function Gamma_i from its own impurity problem; by default pyqcm averages these when
  building the host from the projected lattice Green's function
  (`$PYQCM_ROOT/docs/source/cdmft.rst`, "Subbaths" section; feature since pyqcm > 2.19).

  **This is a genuinely new method, not yet widely used or benchmarked outside this group.** General
  CDMFT intuition doesn't automatically transfer to the multi-system averaging behavior, so read the
  subbath paper and the `cdmft.rst` "Subbaths" section before assuming otherwise. For working code
  (more useful than prose here), see `$PYQCM_ROOT/tests/test_files/test_cdmft_sb.py` (a 2-cluster
  ladder model where one cluster has two bath systems and the other has none, the mixed case) and
  `test_cdmft_1x4_4b_2sb.py` (a single 1x4-chain cluster with two subbath systems, each built from
  its own `cluster_model` with `generators=`/`bath_irrep=True` for a C2 irrep, the symmetry-adapted
  case, showing the `eb{i}_{k}`/`tb{i}_{k}` bath-parameter naming where `k` indexes which subbath
  system a parameter belongs to). Both run automatically under `$PYQCM_ROOT/tests/test_all.py`.

Mixing up which method produced a result changes what conclusions are even valid to draw. A CPT
calculation cannot show a stable symmetry-broken phase (it has no variational or self-consistent
mechanism to select one), so if the user's script produces order anyway, flag it rather than
explaining it away.

## Reading common output quantities

- **Ground state / sector**: `I.ground_state()` reports which target sector actually won; if the
  program had to search several sectors, this confirms which particle number/spin/irrep the physical
  ground state sits in. Worth checking against expectations (e.g. is the filling what was intended?).
- **Cluster/lattice averages** (`cluster_averages`): expectation values of every declared operator.
  For an order parameter, its magnitude and sign are the physical signal; for U or t, this is just a
  consistency check that the parameter took the intended value.
- **Spectral function** (`cluster_spectral_function`, `spectral_function`): the periodized
  one-particle Green's function's imaginary part, i.e. the (C)PT approximation to the ARPES/IPES
  spectrum. Gaps, band structure, and pseudogap-like suppression near the Fermi level are read off
  here. Distinguish the *cluster* spectral function (exact, but for the isolated finite cluster) from
  the *lattice* one (periodized, the actual physical prediction).
- **DoS** (`plot_DoS`): the k-integrated spectral function; a gap here signals an insulating phase, a
  Mott gap in particular if driven by U rather than by band structure. If the calculation *sweeps* a
  parameter (e.g. chemical potential) specifically to probe the Mott transition rather than sit at a
  fixed filling, see the sector-declaration gotcha in `references/practice.md`: the target sectors need
  to track the filling change, not stay fixed at one `N`.
- **Self-energy**: encodes the correlation physics beyond mean-field. Strong frequency dependence
  near zero frequency is the fingerprint of strong correlation effects, as opposed to a self-energy
  that is essentially a constant shift, which is closer to a renormalized band picture.

## Periodization / interpolation schemes

CDMFT (and CPT) results only give you the cluster's real-space Green's function, self-energy, or
cumulant directly; recovering a lattice-periodic, k-dependent quantity from that requires a
periodization (CDMFT) or interpolation (DCA) step. pyqcm exposes several traditional schemes via the
`period=` argument to spectral-function calls (or
`pyqcm.set_global_parameter('periodization', ...)`): `'G'` (periodize the Green's function directly,
the default), `'M'` (periodize the cumulant instead), plus `'S'`, `'C'`, `'N'` variants. See
`$PYQCM_ROOT/docs/source/spectral.rst`, `$PYQCM_ROOT/pyqcm/_spectral.py` docstrings, and `src_qcm/CPT.cpp`, where
this is actually computed (layer 3 in `references/modifying-pyqcm.md`'s architecture).

**Which scheme is "right" is not a solved question in general.** Different choices can disagree
noticeably away from particle-hole symmetry. Don't present a k-resolved feature as a physical
prediction without saying which scheme produced it. Two entries in "Related reading" below cover the
open question (compact tiling, Liouvillian interpolation).

**Liouvillian interpolation is not implemented in pyqcm.** It is an external proposal from a
different group, and implementing and benchmarking it against the existing G/M schemes is a live
direction this group intends to pursue, part of why this skill exists: onboarding new contributors,
including incoming interns, onto this specific piece of forward work. If asked to help implement or
evaluate it, don't assume any existing pyqcm code does this. `CPT.cpp` (layer 3) is the natural place
a periodization scheme lives architecturally, but confirm current status with the maintainers before
treating this as already-existing functionality.

## Conventions you will meet in this group's scripts

Not physics results, just things that look like bugs and aren't:

- **`Up` (oxygen on-site U) unset or zero in an Emery-model script is normally intentional**, not an
  omission, in scripts following the Cu/O decomposition of the St-Cyr & Sénéchal paper below.
- **A cuprate script decomposing into 4 correlated Cu + 8 uncorrelated O + an 8-orbital bath** is
  that paper's decomposition, not an arbitrary choice; check it before "fixing" the geometry.
- **A discontinuity or solver failure partway through an SC dome is not automatically the dome edge.**
  Check the paper for the model in question before reporting a sweep as having found a phase boundary.

## Related reading

Cite these by id rather than summarizing them. **Full texts are not stored in this repo** and no
abstract or excerpt is reproduced here: the papers are under their authors' and publishers'
copyright, not this repo's MIT licence. Fetch what you need locally; `.txt` and `.pdf` under
`references/research/` are git-ignored.

- **pyqcm itself** (canonical description, v2.1): Dionne, Foley, Rousseau & Sénéchal, *Pyqcm: An
  open-source Python library for quantum cluster methods*, SciPost Phys. Codebases 23 (2023),
  [10.21468/SciPostPhysCodeb.23](https://doi.org/10.21468/SciPostPhysCodeb.23).
- **CPT and VCA foundations**: Sénéchal, Pérez & Plouffe, *Cluster Perturbation Theory for Hubbard
  models*, `cond-mat/0205044`.
- **General cluster-method comparisons** (CPT vs CDMFT vs DCA, causality, conservation, broken
  symmetry, susceptibilities), the field's standard review rather than this group's own work: Maier,
  Jarrell, Pruschke & Hettler, *Quantum Cluster Theories*, `cond-mat/0404055`.
- **CDMFT bath fitting and weight functions**: Sénéchal, *Bath optimization in the Cellular DMFT*,
  `1005.1685`. See also `references/practice.md` for the practical weight-function guidance.
- **Subbath CDMFT**: de Lagrave, Sénéchal & Charlebois, *Subbath Cluster Dynamical Mean-Field
  Theory*, `2509.07931`.
- **Periodization, Fermi arcs vs hole pockets, compact tiling**: Verret, Foley, Sénéchal, Tremblay &
  Charlebois, *Fermi arcs vs hole pockets: periodization of a cellular two-band model*, `2107.01344`.
- **Liouvillian interpolation** (proposal, not in pyqcm): Pelz, von Delft & Gleis, *Liouvillian
  interpolation of the self-energy of cluster dynamical mean-field theories*, `2602.16351`.
- **AFM/SC coexistence and competition**, including how much the bath parametrization affects the
  coexistence region: Foley, Verret, Tremblay & Sénéchal, *Coexistence of Superconductivity and
  Antiferromagnetism in the Hubbard model for cuprates*, `1811.12363`.
- **The Emery (three-band) model**, the Cu/O decomposition and per-material hopping parameters this
  group's Emery scripts use: St-Cyr & Sénéchal, *Effect of the Coulomb repulsion and oxygen level on
  charge distribution and superconductivity in the Emery model*, `2503.07810`.
- **Twisted bilayer cuprates, topological superconductivity**: Vibert & Sénéchal, *Topological
  superconductivity in a Hubbard model for twisted bilayer cuprates*, `2605.06923`.
- **Charge-self-consistent CDMFT+DFT for multilayer cuprates**, this group's own work and the entry
  point for both senses of double counting below: Bacq-Labreuil, Lacasse, Tremblay, Sénéchal & Haule,
  *Towards an ab initio theory of high-temperature superconductors: a study of multilayer cuprates*,
  `2410.10019`.
- **DFT+DMFT double counting**: Haule, *Exact double-counting in combining DMFT and DFT*,
  `1501.03438`; Haule, Birol & Kotliar, *Covalency in transition metal oxides within all-electron
  DMFT*, `1310.1158`; Haule, Yee & Kim, *DMFT within the full-potential methods*, `0907.0195`.

**On "double counting"**, two distinct problems share the name, worth separating before reasoning
about either. (1) *DFT+DMFT double counting*: the exchange-correlation functional already contains
part of the local interaction that `U` then adds again, so a potential `V_DC` must be subtracted, and
it is not uniquely defined. Which scheme was used is a load-bearing detail; don't compare numbers
across schemes. (2) *Double counting the local self-energy when nesting cluster DMFT inside a
charge-self-consistent loop*: if the charge self-consistency is driven by single-site DMFT while the
physics of interest comes from a cluster solver, the single-site local self-energy has to be
subtracted from the cluster one or it enters twice. This is the one a pyqcm user is most likely to
meet, since pyqcm supplies the cluster side. Either way, double counting only arises once a model is
derived from or coupled to DFT: a standalone Hubbard/Emery script with hand-chosen parameters has no
double counting problem, and the term is being used loosely if it comes up there.
