# Contributing

The useful contribution here is a **gotcha**: something about pyqcm that cost you real time and is not
written down anywhere else. Pull requests are reviewed before merging, so the bar below is what gets
one accepted quickly rather than a hurdle to clear before opening it.

(For contributing to pyqcm *itself*, the library, this is the wrong repo. See
`skills/pyqcm-skill/references/modifying-pyqcm.md` and the upstream project.)

## The bar for a new gotcha

All four must hold:

- **General.** It would have changed the outcome for someone other than you, on a different model or
  system. A fix specific to your Hamiltonian is not a gotcha.
- **Durable.** It is a property of pyqcm or of the method, not of one script, one dataset, or one
  parameter choice. If the next pyqcm release makes it false, say so and name the version.
- **New.** Not already in `references/practice.md` and not already in the upstream documentation. Check
  both.
- **It cost something.** Wasted hours, a wrong result, a discarded run, or an error message that sent
  you the wrong way. Merely interesting is not enough.

## Format

New entries go in `skills/pyqcm-skill/references/practice.md`. Keep them short and concrete: that file
is meant to be skimmed, not read end to end. Use this shape:

```
## <short title>

What: <the surprising behavior or mistake, stated plainly>
Why it happens: <the underlying reason, if known>
What to do instead: <the fix or the correct convention>
```

Name pyqcm versions explicitly when behaviour depends on them, and flag anything that reflects a
maintainer's choice that could change upstream rather than a fixed property of the method.

## Everything else

Corrections, clearer wording, and updates when upstream changes are all welcome. Two house rules from
`references/guidelines.md`.

Do not add pyqcm source code to this repository. pyqcm is GPL-3.0-or-later and this repo is MIT;
describe behaviour in your own words instead of pasting it.
