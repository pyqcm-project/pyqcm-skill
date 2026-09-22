# Research paper citations

**This repo cites papers; it does not redistribute them.** No paper full text is tracked here,
regardless of its licence, its publisher, or who wrote it. `skills/pyqcm-skill/references/research/**/*.txt` and
`*.pdf` are git-ignored, so a local copy you fetch stays local.

The skill's reference files cite papers by arXiv id or DOI. Use this table to resolve one, then
fetch it if you need the full text.

## Fetching a local copy

```bash
# one paper
curl -L -o skills/pyqcm-skill/references/research/periodization/2602.16351.pdf https://arxiv.org/pdf/2602.16351

# all arXiv-hosted papers below
while read -r id dir; do
  curl -L -o "skills/pyqcm-skill/references/research/$dir/$(echo "$id" | tr / _).pdf" "https://arxiv.org/pdf/$id"
done <<'EOF'
cond-mat/0205044 quantum_cluster_methods
cond-mat/0404055 quantum_cluster_methods
1005.1685        quantum_cluster_methods
2509.07931       quantum_cluster_methods
2107.01344       periodization
2602.16351       periodization
1811.12363       cuprates
2410.10019       cuprates
2503.07810       cuprates
2605.06923       cuprates
0907.0195        double_counting
1310.1158        double_counting
1501.03438       double_counting
EOF
```

## Quantum cluster methods

| Paper | arXiv | DOI |
|---|---|---|
| Dionne, Foley, Rousseau & Sénéchal, *Pyqcm: An open-source Python library for quantum cluster methods*, SciPost Phys. Codebases 23 (2023) | none | [10.21468/SciPostPhysCodeb.23](https://doi.org/10.21468/SciPostPhysCodeb.23) |
| Sénéchal, Pérez & Plouffe, *Cluster Perturbation Theory for Hubbard models*, PRB 66, 075129 | `cond-mat/0205044` | [10.1103/PhysRevB.66.075129](https://doi.org/10.1103/PhysRevB.66.075129) |
| Maier, Jarrell, Pruschke & Hettler, *Quantum Cluster Theories*, RMP 77, 1027 (2005) | `cond-mat/0404055` | [10.1103/RevModPhys.77.1027](https://doi.org/10.1103/RevModPhys.77.1027) |
| Sénéchal, *Bath optimization in the Cellular DMFT*, PRB 81, 235125 (2010) | `1005.1685` | [10.1103/PhysRevB.81.235125](https://doi.org/10.1103/PhysRevB.81.235125) |
| de Lagrave, Sénéchal & Charlebois, *Subbath Cluster Dynamical Mean-Field Theory* | `2509.07931` | preprint, none as of 2026-09-09 |

## Periodization / interpolation

| Paper | arXiv | DOI |
|---|---|---|
| Verret, Foley, Sénéchal, Tremblay & Charlebois, *Fermi arcs vs hole pockets: periodization of a cellular two-band model*, PRB 105, 035117 | `2107.01344` | [10.1103/PhysRevB.105.035117](https://doi.org/10.1103/PhysRevB.105.035117) |
| Pelz, von Delft & Gleis, *Liouvillian interpolation of the self-energy of cluster dynamical mean-field theories* | `2602.16351` | preprint, none as of 2026-09-09 |

## Cuprates

| Paper | arXiv | DOI |
|---|---|---|
| Foley, Verret, Tremblay & Sénéchal, *Coexistence of Superconductivity and Antiferromagnetism in the Hubbard model for cuprates*, PRB 99, 184510 (2019) | `1811.12363` | [10.1103/PhysRevB.99.184510](https://doi.org/10.1103/PhysRevB.99.184510) |
| St-Cyr & Sénéchal, *Effect of the Coulomb repulsion and oxygen level on charge distribution and superconductivity in the Emery model*, SciPost Phys. Core 8, 043 (2025) | `2503.07810` | [10.21468/SciPostPhysCore.8.2.043](https://doi.org/10.21468/SciPostPhysCore.8.2.043) |
| Vibert & Sénéchal, *Topological superconductivity in a Hubbard model for twisted bilayer cuprates* | `2605.06923` | [10.1103/hdxf-sqwn](https://doi.org/10.1103/hdxf-sqwn) |
| Bacq-Labreuil, Lacasse, Tremblay, Sénéchal & Haule, *Towards an ab initio theory of high-temperature superconductors: a study of multilayer cuprates* | `2410.10019` | preprint, none as of 2026-09-09 |

## Double counting

| Paper | arXiv | DOI |
|---|---|---|
| Haule, *Exact double-counting in combining DMFT and DFT*, PRL 115, 196403 (2015) | `1501.03438` | [10.1103/PhysRevLett.115.196403](https://doi.org/10.1103/PhysRevLett.115.196403) |
| Haule, Birol & Kotliar, *Covalency in transition metal oxides within all-electron DMFT*, PRB 90, 075136 | `1310.1158` | [10.1103/PhysRevB.90.075136](https://doi.org/10.1103/PhysRevB.90.075136) |
| Haule, Yee & Kim, *DMFT within the full-potential methods*, PRB 81, 195107 (2010) | `0907.0195` | [10.1103/PhysRevB.81.195107](https://doi.org/10.1103/PhysRevB.81.195107) |

The Bacq-Labreuil paper is relevant to both cuprates and double counting; it is listed once, under
cuprates.
