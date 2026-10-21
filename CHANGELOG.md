# Changelog

## 1.0.1 (2026-10-21)

- `HARNESS.md`: the no em dash rule now says that the normal dash on the keyboard (the hyphen-minus) is fine wherever it is needed (a compound word, a range, a list marker, a command flag, a spaced hyphen in docs and comments, an empty value in a user interface), not only in docs and comments. The code check flags only the em dash and the en dash.
- `docs/code-check.md`, `README.md`: the text rules step and the trust note say the same.

## 1.0.0 (2026-10-21)

First release.

- `HARNESS.md`: the entry file with the working rules (code in git first, version on every deploy by the size of the change, cache busting by file name, no workflow run without the owner's go, commits by the project's author only, no em dash, secrets never appear, lessons with evidence), the decisions before coding, the environment with the best tools for the job, the code check gate, testing, security posture, shipping, parity across copies, lessons with evidence, and how the agent builds the project's runner.
- `docs/code-check.md`: the seven gate steps, the two rules that never let a check pass silently, changed-files mode, reviewed false positives, the linter policy, enforcement by the build scripts.
- `docs/testing.md`: three tiers, path filters, isolated suites, lanes, one rerun rule, test selection by import graph, preflight with fixes, a fresh database per run, FAILED against BLOCKED, flags, the A/B rule.
- `docs/shipping.md`: the repository and its remote (a new repository, choosing a host, git on Windows, sessions on a cloud sandbox), the ten ship steps with main as production, pull request text, commit authorship, dependency update bot pull requests, workflow runs on the host, the emergency fallback.
- `docs/security.md`: passive, light and active checks of the deployed surface, the rules before any check, the defensive design they assume.
- `docs/lessons.md`: cue, lesson, evidence; memory, snapshot and inbox.
- `docs/playbooks.md`, `docs/playbooks/3d-renders.md`, `docs/playbooks/interactive-web.md`: decisions before coding, WordPress, 3D renders from an image, interactive web pages.
- `docs/tools-by-language.md`: the tool table per language and the cross-language tools.
- `README.md`: what a harness is made of, install, first use, files, trust, versioning.
