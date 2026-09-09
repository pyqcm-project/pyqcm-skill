# Interpreting pyqcm physics results

Don't treat pyqcm output as opaque numbers to report back. Connect it to what the method is actually
approximating: get the mapping between *output quantity* and *physical meaning* right, and ground
non-obvious claims in the papers listed in `references/research/CITATIONS.md` rather than in general
intuition about Hubbard models, since the specifics of CPT/VCA/CDMFT matter.

## Which method is being used, and what that implies

- **CPT** (Cluster Perturbation Theory): exact-diagonalizes isolated clusters, then reconnects them
  perturbatively (strong-coupling perturbation theory in the inter-cluster hopping). No
  self-consistency loop. Good for spectral properties (ARPES-like spectral function, DOS) at fixed
  parameters; cannot capture symmetry-broken order on its own.
- **VCA** (Variational Cluster Approach): adds fictitious symmetry-breaking (or bath) terms to the
  cluster Hamiltonian as variational parameters, and extremizes the Potthoff self-energy functional
  over them. A stationary point (not necessarily a minimum in every direction, so check which
  parameters are saddle-point vs. minimum) signals the true value of that variational parameter; its
  magnitude away from zero is the order parameter. See `pyqcm/docs/source/vca.rst` and Sénéchal et
  al. 2002 ("Cluster perturbation theory for Hubbard models",
  `arXiv:cond-mat/0205044`) for the CPT/VCA foundations.
- **CDMFT** (Cellular/Cluster DMFT): clusters are coupled to a bath of uncorrelated orbitals whose
  parameters are tuned so the cluster's hybridization function best reproduces the lattice
  self-consistency condition, iterated to convergence. This captures local dynamical correlations
  (unlike VCA/CPT) at the cost of the bath-fitting approximation. See `pyqcm/docs/source/cdmft.rst`.
- **Subbath CDMFT**: a variant published this year by this group (de Lagrave, Sénéchal &
  Charlebois; `arXiv:2509.07931`), where a cluster is
  associated with *more than one* independent bath system instead of one. In pyqcm this is not a
  special mode or class: it is implemented by passing a **list** of `cluster_model` objects (instead
  of a single one) to `pyqcm.cluster()`, each with its own bath. Each such "system" gets its own
  self-energy Sigma_i and hybridization function Gamma_i from its own impurity problem; by default
  pyqcm averages these when building the host from the projected lattice Green's function, giving a
  tighter effective correspondence between host and impurity models than a single larger bath would
  at the same cost (`pyqcm/docs/source/cdmft.rst`, "Subbaths" section; feature since pyqcm > 2.19).
  Physically this buys an extended bath representation (better hybridization fit, e.g. sharper Mott
  gap reproduction, see the subbath paper's Fig. 8/10 comparisons) at a fraction of the ED cost of
  one large bath, because each subbath is diagonalized as its own smaller impurity problem.

  **This is a genuinely new method (2026), not yet widely used or benchmarked outside this group.**
  General CDMFT intuition doesn't automatically transfer to the multi-system averaging behavior, so
  read the subbath paper and the `cdmft.rst` "Subbaths" section before assuming otherwise. For
  working code (more useful than prose here), see `pyqcm/tests/test_files/test_cdmft_sb.py` (a
  2-cluster ladder model where one cluster has two bath systems and the other has none, the mixed
  case) and `test_cdmft_1x4_4b_2sb.py` (a single 1x4-chain cluster with two subbath systems, each
  built from its own `cluster_model` with `generators=`/`bath_irrep=True` for a C2 irrep, the
  symmetry-adapted case, showing the `eb{i}_{k}`/`tb{i}_{k}` bath-parameter naming where `k` indexes
  which subbath system a parameter belongs to). Both run automatically under `pyqcm/tests/test_all.py`.

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
  fixed filling, see the sector-declaration gotcha in `references/gotchas.md`: the target sectors need
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
`pyqcm/docs/source/spectral.rst`, `pyqcm/pyqcm/_spectral.py` docstrings, and `src_qcm/CPT.cpp`, where
this is actually computed (layer 3 in `references/contributing.md`'s architecture). Which scheme is
"right" is not a solved question in general: different choices can disagree noticeably away from
particle-hole symmetry, and traditional interpolation of a frequency-dependent quantity can violate
causality or wash out fine Fermi-surface structure such as hole pockets.

**Precursor, compact tiling:** Verret, Foley, Sénéchal, Tremblay & Charlebois,
`arXiv:2107.01344`, introduce a "compact tiling" periodization
scheme for a two-band cellular model, alongside the traditional G/M schemes, specifically to settle
whether the low-doping cuprate Fermi surface is hole pockets or disconnected Fermi arcs. Compact
tiling reconstructs a k-dependent quantity by tiling short-ranged real-space hoppings and anomalous
terms directly rather than periodizing G, Sigma, or M in the traditional sense; it works best when
hoppings are short-ranged and degrades if used to rebuild longer-ranged structure. This is the direct
conceptual precursor to the Liouvillian interpolation scheme below (same underlying goal: resolving
Fermi arcs vs. hole pockets by getting periodization right), so read it first for context on why this
is a long-standing open question before jumping to the 2026 L-interpolation proposal.

**Emerging alternative, Liouvillian interpolation (L-interpolation):** a February 2026 paper (Pelz,
von Delft & Gleis, `arXiv:2602.16351`) proposes interpolating the
frequency-*independent* matrix elements of the self-energy's continued-fraction expansion (Liouvillian
matrix elements) instead of interpolating G, Sigma, or M directly. This is argued to have a more local
Fourier expansion than traditional Q-space interpolation and to inherently conserve causality, with
demonstrated improvements resolving Fermi/Luttinger arcs that traditional periodization misses.
**This is not yet implemented in pyqcm.** It is an external result from a different group (LMU Munich
/ Rutgers), and implementing and benchmarking it against the existing G/M schemes is a live direction
this group intends to pursue, part of why this skill exists: onboarding new contributors, including
incoming interns, onto this specific piece of forward work. If asked to help implement or evaluate
it, don't assume any existing pyqcm code does this. `CPT.cpp` (layer 3) is the natural place a
periodization scheme lives architecturally, but confirm current status with Antoine before treating
this as already-existing functionality.

## Grounding claims in the literature

`references/research/CITATIONS.md` lists the papers this skill cites, from the method's foundations
to applications this group cares about, grouped into four topics: quantum cluster methods (CPT/VCA/
CDMFT foundations and pyqcm itself), periodization and interpolation schemes, cuprates (multilayer
cuprates, oxygen level and charge distribution effects on superconductivity, twisted bilayer
cuprates, topological superconductivity), and double counting (see the dedicated entry below). The
full texts are not stored in this repo; the manifest carries arXiv ids, DOIs, and a fetch command.

For general quantum-cluster-theory questions not specific to any one application (comparing
CPT/CDMFT/DCA on general grounds, causality and conservation properties of cluster methods,
broken-symmetry phases, susceptibilities), start from Maier, Jarrell, Pruschke & Hettler's "Quantum
Cluster Theories" review (`arXiv:cond-mat/0404055`, Rev. Mod. Phys. 77, 1027 (2005)) rather than the more narrowly-scoped papers below;
it is the field's standard reference review, not this group's own work. When a result plausibly connects to one of the applications below (e.g. the physics of a
cuprate-like multi-orbital model, or anything involving oxygen p-orbitals and charge transfer), check
the relevant paper rather than reasoning purely from generic single-band Hubbard intuition: cuprates
specifically require the charge-transfer-insulator picture, not the plain Mott picture.

The SciPost Codebase paper (`SciPost Phys. Codebases 23`) is
the canonical description of pyqcm itself (v2.1) and the right reference point for "how does pyqcm
implement X" questions that are about the method rather than the code. For Mott transition physics
specifically (chemical-potential-driven filling changes, the Mott gap), both the subbath paper
(`arXiv:2509.07931`, Fig. 7 and around) and the SciPost
Codebase paper's 1D Hubbard example discuss it directly; check these before reasoning about Mott
transition behavior from generic intuition.

For **AFM/SC coexistence and competition** specifically (the two dominant, competing symmetry-broken
phases in the cuprate Hubbard model: AFM breaks SO(3) with order parameter M, d-wave SC breaks U(1)
with order parameter Delta), see Foley, Verret, Tremblay & Sénéchal,
`arXiv:1811.12363`. Key points before reasoning about a coexistence
result from this group's models: microscopic (spatially homogeneous) coexistence is distinct from
macroscopic coexistence arising from inhomogeneity or thermodynamic phase separation, and its clean
signature is a nonzero "u-triplet" order parameter, nonzero only if both M and Delta are nonzero and
itself a kind of pair-density wave, though a different one from the PDW seen in STM experiments. The
paper also shows that the bath parametrization used matters quantitatively: a more complete or
general parametrization (maximal freedom in the hybridization function per bath orbital) shrinks the
coexistence region found by earlier, more restrictive parametrizations. Don't treat a coexistence
region as parametrization-independent; check what bath parametrization was used before comparing
results across scripts or papers.

For **the Emery (three-band) model specifically**, meaning any script decomposing a cuprate lattice
into a correlated Cu cluster plus an uncorrelated O cluster, St-Cyr & Sénéchal (`arXiv:2503.07810`,
SciPost Phys. Core 8, 043 (2025)) is this group's own primary reference and normally the first thing
to check, not just background. It is the source of the 4-correlated-Cu +
8-uncorrelated-O + 8-orbital-bath CDMFT decomposition this group's Emery scripts use, of Table I's
hopping ratios `tpd/tpp`, `t'pp/tpp`, `(eps_p - eps_d)/tpp` for BSCCO/LSCO/YBCO/NCCO (derived from
DFT, Refs [15,16] therein), and of NMR-constrained `U - eps_p` estimates per material (YBCO about 6,
LSCO about 10). `Up` (oxygen on-site U) is deliberately neglected throughout (DFT-justified: oxygens
near-filled, `Up` small), so a zero or unset `Up` in a script matching this decomposition is normally
intentional, not a bug.

Two results from that paper matter for interpreting a doping or mu sweep of the SC order parameter:

1. The computed d-wave dome is not a single smooth curve. Fig. 3 shows a *discontinuity within the SC
   region itself* at "optimal doping", the boundary between SC-with-pseudogap and SC-without, so a
   solver failure or sudden jump there is not necessarily the dome edge.
2. Those dome calculations explicitly ignore antiferromagnetism near half-filling (no AFM order
   parameter in that part of the paper), so an SC value found close to half-filling by a similarly
   AFM-free script may be a sector-restricted result rather than the true ground state. Cross-check
   against the AFM/SC coexistence paper above if that matters for the result being reported.

For **double counting**, meaning what to subtract when a correlated model is embedded in a
band-structure calculation, see the double counting entries in `references/research/CITATIONS.md`.
This only arises once a model is derived from or coupled to DFT; a standalone Hubbard/Emery
script with hand-chosen parameters has no double counting problem, and the term is being used loosely
if it comes up there. Two genuinely distinct meanings are at play, worth separating before reasoning
about either:

1. **The DFT+DMFT double counting.** The exchange-correlation functional already contains part of the
   local interaction that `U` then adds again, so a potential `V_DC` must be subtracted. It is not
   uniquely defined. Three schemes matter here: the widely used **fully-localized-limit (FLL)**, the
   **nominal** one (`V_DC` fixed by a chosen integer occupancy rather than the self-consistent `n_d`),
   and Haule's **exact** double counting derived from a continuum representation of DMFT
   (`arXiv:1501.03438`, PRL 115, 196403). The headline result is that nominal is much closer to exact
   than FLL is, so FLL numbers and nominal numbers are not interchangeable, and a scheme change alone
   can move the charge-transfer energy and the p-d splitting. Haule, Birol & Kotliar
   (`arXiv:1310.1158`, PRB 90, 075136) is the companion on covalency in transition-metal oxides and
   shows how strongly the choice feeds through to `n_d`; Haule, Yee & Kim (`arXiv:0907.0195`, PRB 81,
   195107) is the implementation paper for charge-self-consistent DFT+DMFT in full-potential methods.
2. **Double counting the local self-energy when nesting cluster DMFT inside a charge-self-consistent
   loop.** Distinct problem, same name. If the charge self-consistency is driven by single-site DMFT
   while the physics of interest comes from a cluster solver, the single-site local self-energy has to
   be subtracted from the cluster one or it enters twice. This is the one a pyqcm user is most likely
   to meet in practice, since pyqcm supplies the cluster side.

Bacq-Labreuil, Lacasse, Tremblay, Sénéchal & Haule (`arXiv:2410.10019`) is the paper to start from:
it is this group's own work, uses both notions, and is the reference for the charge-self-consistent
CDMFT+DFT route to material-specific cuprate predictions. Its physical result, Tc growing from
single- to tri-layer compounds via a reduced charge-transfer gap and hence larger superexchange `J`,
is also the cleanest available demonstration that the charge-transfer gap, not a plain Mott gap, is
the controlling scale in these materials.
