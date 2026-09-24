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

Update later with `/plugin update pyqcm-skill`. For the claude.ai web app instead, download a zip of the repo and upload it under
Settings > Capabilities > Skills. The skill ships **no pyqcm source**. Most of what it knows refers to a pyqcm checkout, so tell it
where yours is:

```bash
export PYQCM_ROOT=/path/to/pyqcm  # the repo root, the directory containing src_ed/
```

Without it, the skill tries to derive the path from an installed `pyqcm` package and falls back to
asking you. If you have no checkout yet:

```bash
git clone https://github.com/pyqcm-project/pyqcm.git
```

## Contents

The entry point is [`skills/pyqcm-skill/SKILL.md`](skills/pyqcm-skill/SKILL.md). It resolves
`$PYQCM_ROOT`, routes each kind of question to the right reference, and lists every file under
`references/`.

## Relationship to Pyqcm

This repository contains **no pyqcm source code**. pyqcm is authored by David Senechal and licensed
GPL-3.0-or-later; see https://github.com/pyqcm-project/pyqcm.

It also contains **no paper text**. Cited work stays under its authors' and
publishers' copyright, so the skill cites by arXiv id only. Copies you
fetch into `skills/pyqcm-skill/references/research/` (create it; it ships empty) are git-ignored and
stay local.

## Contributing

Gotchas are the most valuable thing you can send. See [CONTRIBUTING.md](CONTRIBUTING.md).
