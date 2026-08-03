# Interpreting pyqcm physics results

Don't treat pyqcm output as opaque numbers to report back — connect it to what the method is
actually approximating. Get the mapping between *output quantity* and *physical meaning* right,
and ground non-obvious claims in the papers under `references/research/` rather than general
intuition about Hubbard models, since the specifics of CPT/VCA/CDMFT matter.

## Which method is being used, and what that implies

- **CPT** (Cluster Perturbation Theory) — exact-diagonalizes isolated clusters, then reconnects them
  perturbatively (strong-coupling perturbation theory in the inter-cluster hopping). No
  self-consistency loop. Good for spectral properties (ARPES-like spectral function, DOS) at fixed
  parameters; cannot capture symmetry-broken order on its own.
- **VCA** (Variational Cluster Approach) — adds fictitious symmetry-breaking (or bath) terms to the
  cluster Hamiltonian as variational parameters, and extremizes the Potthoff self-energy functional
  over them. A stationary point (not necessarily a minimum in every direction — check which
  parameters are saddle-point vs. minimum) signals the true value of that variational parameter;
  its magnitude away from zero is the order parameter. See `pyqcm/docs/source/vca.rst` and
  Sénéchal et al. 2002 ("Cluster perturbation theory for Hubbard models", `references/research/
  quantum_cluster_methods/0205044v1.txt`) for the CPT/VCA foundations.
- **CDMFT** (Cellular/Cluster DMFT) — clusters are coupled to a bath of uncorrelated orbitals whose
  parameters are tuned so the cluster's hybridization function best reproduces the lattice
  self-consistency condition, iterated to convergence. This captures local dynamical correlations
  (unlike VCA/CPT) at the cost of the bath-fitting approximation. See `pyqcm/docs/source/cdmft.rst`.
- **Subbath CDMFT** — a variant, published this year by this group (de Lagrave, Sénéchal &
  Charlebois; `references/research/quantum_cluster_methods/2509.07931v2.txt`), where a cluster is associated with *more than
  one* independent bath system instead of one. In pyqcm this is not a special mode or class — it's
  implemented simply by passing a **list** of `cluster_model` objects (instead of a single one) to
  `pyqcm.cluster()`, each with its own bath. Each such "system" gets its own self-energy Σᵢ and
  hybridization function Γᵢ from its own impurity problem; by default pyqcm averages these when
  building the host from the projected lattice Green's function, giving a tighter effective
  correspondence between host and impurity models than a single larger bath would at the same cost
  (`pyqcm/docs/source/cdmft.rst`, "Subbaths" section; feature since pyqcm > 2.19). Physically, this
  buys an extended bath representation (better hybridization fit, e.g. sharper Mott gap reproduction,
  see the subbath paper's Fig. 8/10 comparisons) at a fraction of the ED cost of one large bath,
  because each subbath is diagonalized as its own smaller impurity problem rather than one big one.
  **This is a genuinely new method (2026), not yet widely used or benchmarked outside this group** —
  one of the reasons this skill exists is to help new group members (including future interns) pick
  it up correctly and actually use/benchmark it rather than defaulting to single-bath CDMFT out of
  habit. General CDMFT intuition doesn't automatically transfer to the multi-system averaging
  behavior — read the subbath paper and the `cdmft.rst` "Subbaths" section before assuming otherwise.
  For working code (more useful than prose here), see `pyqcm/tests/test_files/test_cdmft_sb.py` (a
  2-cluster ladder model where one cluster has two bath systems and the other has none — shows the
  mixed case) and `test_cdmft_1x4_4b_2sb.py` (a single 1×4-chain cluster with two subbath systems, each
  built from its own `cluster_model` with `generators=`/`bath_irrep=True` for a C2 irrep — shows the
  symmetry-adapted case and the `eb{i}_{k}`/`tb{i}_{k}` bath-parameter naming where `k` indexes which
  subbath system a parameter belongs to). Both are run automatically by `pyqcm/tests/test_all.py`.

Mixing up which method produced a result changes what conclusions are even valid to draw — e.g. a
CPT calculation cannot show a stable symmetry-broken phase (it has no variational/self-consistent
mechanism to select one); if the user's script produces order anyway, that's worth flagging rather
than explaining away.

## Reading common output quantities

- **Ground state / sector** — `I.ground_state()` reports which target sector actually won; if the
  program had to search several sectors, this confirms which particle number/spin/irrep the physical
  ground state sits in. Worth checking against expectations (e.g. is the filling what was intended?).
- **Cluster/lattice averages** (`cluster_averages`) — expectation values of every declared operator.
  For an order parameter, its magnitude and sign are the physical signal; for U or t, this is just a
  consistency check that the parameter took the intended value.
- **Spectral function** (`cluster_spectral_function`, `spectral_function`) — the periodized one-
  particle Green's function's imaginary part, i.e. the (C)PT approximation to the ARPES/IPES
  spectrum. Gaps, band structure, and pseudogap-like suppression near the Fermi level are read off
  here. Distinguish the *cluster* spectral function (exact, but for the isolated finite cluster) from
  the *lattice* one (periodized, the actual physical prediction).
- **DoS** (`plot_DoS`) — the k-integrated spectral function; a gap here signals an insulating phase,
  a Mott gap in particular if driven by U rather than by band structure. If the calculation *sweeps*
  a parameter (e.g. chemical potential) specifically to probe the Mott transition rather than sit at a
  fixed filling, see the sector-declaration gotcha in `references/gotchas.md` — the target sectors need
  to track the filling change, not stay fixed at one `N`.
- **Self-energy** — encodes the correlation physics beyond mean-field; a self-energy with strong
  frequency dependence near zero frequency is the fingerprint of strong correlation effects (as
  opposed to a self-energy that's essentially a constant shift, which is closer to a renormalized
  band picture).

## Periodization / interpolation schemes

CDMFT (and CPT) results only give you the cluster's real-space Green's function/self-energy/cumulant
directly; recovering a lattice-periodic, k-dependent quantity from that requires a periodization
(CDMFT)/interpolation (DCA) step. pyqcm exposes several traditional schemes via the `period=` argument
to spectral-function calls (or `pyqcm.set_global_parameter('periodization', ...)`): `'G'` (periodize
the Green's function directly — the default), `'M'` (periodize the cumulant instead), plus `'S'`,
`'C'`, `'N'` variants — see `pyqcm/docs/source/spectral.rst`, `pyqcm/pyqcm/_spectral.py` docstrings,
and `src_qcm/CPT.cpp` (where this is actually computed, layer 3 in `references/contributing.md`'s
architecture). Which scheme is "right" is not a solved question in general — different choices can
disagree noticeably away from particle-hole symmetry, and traditional interpolation of a
frequency-dependent quantity can violate causality or wash out fine Fermi-surface structure (e.g.
hole pockets).

**Precursor — compact tiling:** Verret, Foley, Sénéchal, Tremblay & Charlebois,
`references/research/periodization/2107.01344v1.txt`, introduce a "compact tiling" periodization scheme for a
two-band cellular model, alongside the traditional G/M schemes, specifically to settle whether the
low-doping cuprate Fermi surface is hole pockets or disconnected Fermi arcs. Compact tiling
reconstructs a k-dependent quantity by tiling short-ranged real-space hoppings/anomalous terms
directly rather than periodizing G, Σ, or M in the traditional sense — it works best when hoppings
are short-ranged and degrades if used to try to rebuild longer-ranged structure. This is the direct
conceptual precursor to the Liouvillian interpolation scheme below (same underlying goal: resolving
Fermi arcs vs. hole pockets by getting periodization right), so read it first for context on why this
is a long-standing open question in the field before jumping to the 2026 paper's L-interpolation
proposal.

**Emerging alternative — Liouvillian interpolation (L-interpolation):** a February 2026 paper (Pelz,
von Delft & Gleis, `references/research/periodization/2602.16351v1.txt`) proposes interpolating the
frequency-*independent* matrix elements of the self-energy's continued-fraction expansion (Liouvillian
matrix elements) instead of interpolating G, Σ, or M directly — this is argued to have a more local
Fourier expansion than traditional Q-space interpolation and to inherently conserve causality, with
demonstrated improvements resolving Fermi/Luttinger arcs that traditional periodization misses. **This
is not yet implemented in pyqcm** — it's an external result from a different group (LMU Munich /
Rutgers), and implementing/benchmarking it against the existing G/M schemes is a live direction this
group intends to pursue (part of why this skill exists — to onboard new contributors, including
incoming interns, onto this specific piece of forward work). If asked to help implement or evaluate
it, don't assume any existing pyqcm code does this — `CPT.cpp` (layer 3) is the natural place a
periodization scheme lives architecturally, but confirm current status with Antoine before treating
this as already-existing functionality.

## Grounding claims in the literature

`references/research/` holds papers spanning the method's foundations to applications this group
cares about, sorted into three topic subfolders: `quantum_cluster_methods/` (CPT/VCA/CDMFT
foundations and pyqcm itself), `periodization/` (periodization/interpolation schemes), and
`cuprates/` (multilayer cuprates, oxygen level/charge distribution effects on superconductivity,
twisted bilayer cuprates, topological superconductivity). For general quantum-cluster-theory
questions not specific to any one application — comparing CPT/CDMFT/DCA on general grounds,
causality/conservation properties of cluster methods, broken-symmetry phases, susceptibilities — start
from Maier, Jarrell, Pruschke & Hettler's "Quantum Cluster Theories" review (`references/research/
quantum_cluster_methods/0404055v1.txt`) rather than the more narrowly-scoped papers below;
it's the field's standard reference review, not this group's own work. When a result plausibly
connects to one of
these — e.g. the physics of a cuprate-like multi-orbital model, or anything involving oxygen
p-orbitals and charge transfer — check the relevant paper rather than reasoning purely from the
generic single-band Hubbard intuition; cuprates specifically require the charge-transfer-insulator
picture, not the plain Mott picture. The SciPost Codebase paper
(`references/research/quantum_cluster_methods/SciPostPhysCodeb_23.txt`) is the
canonical description of pyqcm itself (v2.1) and is the right citation/reference point for "how does
pyqcm implement X" questions that are about the method rather than the code. For Mott transition
physics specifically (chemical-potential-driven filling changes, the Mott gap), both the subbath paper
(`references/research/quantum_cluster_methods/2509.07931v2.txt`, Fig. 7 and around) and the SciPost Codebase paper's 1D Hubbard example discuss it
directly — check these before reasoning about Mott transition behavior from generic intuition.

For **AFM/SC coexistence and competition** specifically (the two dominant, competing symmetry-broken
phases in the cuprate Hubbard model — AFM breaks SO(3) with order parameter M, d-wave SC breaks U(1)
with order parameter Delta), see Foley, Verret, Tremblay & Sénéchal, `references/research/
cuprates/1811.12363v2.txt`. Key points worth knowing before reasoning about a coexistence result from this
group's models: microscopic (spatially homogeneous) coexistence is distinct from macroscopic
coexistence arising from inhomogeneity or thermodynamic phase separation, and its clean signature is
a nonzero "u-triplet" order parameter (nonzero only if both M and Delta are nonzero — itself a kind of
pair-density wave, though a different one from the PDW seen in STM experiments). The paper also shows
the bath parametrization used matters quantitatively for this result: a more complete/general
parametrization (maximal freedom in the hybridization function per bath orbital) shrinks the
coexistence region found by earlier, more restrictive parametrizations. Don't treat a coexistence
region as parametrization-independent — check what bath parametrization was used before comparing
results across scripts or papers.

For **the Emery (three-band) model specifically** — any script decomposing a cuprate lattice into a
correlated Cu cluster plus an uncorrelated O cluster — St-Cyr & Sénéchal,
`references/research/cuprates/St-Cyr and Sénéchal - 2025 - Effect of the Coulomb repulsion and oxygen
level on charge distribution and superconductivity in the.txt`, is this group's own primary reference
and normally the first thing to check, not just background: it's the source of the 4-correlated-Cu +
8-uncorrelated-O + 8-orbital-bath CDMFT decomposition this group's Emery scripts use, Table I's
hopping ratios `tpd/tpp`, `t'pp/tpp`, `(εp − εd)/tpp` for BSCCO/LSCO/YBCO/NCCO (derived from DFT,
Refs [15,16] therein), and NMR-constrained `U − εp` estimates per material (YBCO ≈ 6, LSCO ≈ 10).
`Up` (oxygen on-site U) is deliberately neglected throughout (DFT-justified: oxygens near-filled, Up
small) — a zero/unset `Up` in a script matching this decomposition is normally intentional, not a bug.
Two results from this paper matter for interpreting a doping/mu sweep of the SC order parameter: (1)
the computed d-wave dome is not a single smooth curve — Fig. 3 shows a *discontinuity within the SC
region itself* at "optimal doping," the boundary between SC-with-pseudogap and SC-without, so a
solver failure or a sudden jump there is not necessarily the dome edge; (2) those dome calculations
explicitly ignore antiferromagnetism near half-filling (no AFM order parameter in that part of the
paper), so an SC value found close to half-filling by a similarly AFM-free script may be a
sector-restricted result rather than the true ground state — cross-check against the AFM/SC
coexistence paper above if that matters for the result being reported.
