# pyqcm-skill

A Claude Code skill for [pyqcm](https://github.com/pyqcm-project/pyqcm), a library implementing
quantum cluster methods (CPT, VCA, CDMFT) for strongly correlated electron models such as the Hubbard
model.

It can help with four things: getting pyqcm installed, writing and debugging pyqcm scripts,
interpreting what the outputs physically mean, and contributing to the project.

## Install

```
/plugin marketplace add pyqcm-project/pyqcm-skill
/plugin install pyqcm-skill@pyqcm-skill
```

Update later with `/plugin update pyqcm-skill`.

For the claude.ai web app instead, download a zip of the repo and upload it under
Settings > Capabilities > Skills.

## Point it at your pyqcm checkout

The skill ships **no pyqcm source**. Most of what it knows refers to a pyqcm checkout, so tell it
where yours is:

```bash
export PYQCM_ROOT=/path/to/pyqcm     # the repo root, the directory containing src_ed/
```

Without it, the skill tries to derive the path from an installed `pyqcm` package and falls back to
asking you. If you have no checkout yet:

```bash
git clone https://github.com/pyqcm-project/pyqcm.git
```

## What is in here

| Path | Contents |
|---|---|
| `skills/pyqcm-skill/SKILL.md` | Job routing and the `$PYQCM_ROOT` resolution |
| `references/practice.md` | The maintainer's working notes: sharp edges, conventions, and mistakes worth not repeating |
| `references/install.md` | Getting a working build |
| `references/scripting.md` | Writing and debugging pyqcm scripts |
| `references/physics.md` | Reading pyqcm output as physics |
| `references/modifying-pyqcm.md` | Layer boundaries in the C++ and Python source |
| `references/bath-parametrizer.md` | Symmetry-constrained CDMFT bath parametrization |
| `references/hpc.md` | Module-based HPC clusters |
| `references/research/CITATIONS.md` | Cited papers by arXiv id and DOI |

## Relationship to pyqcm

This repository contains **no pyqcm source code**. pyqcm is authored by David Senechal and licensed
GPL-3.0-or-later; see https://github.com/pyqcm-project/pyqcm.

## Contributing

Gotchas are the most valuable thing you can send. See [CONTRIBUTING.md](CONTRIBUTING.md).
