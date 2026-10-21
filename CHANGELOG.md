# Changelog

## 1.3.0 (2026-10-21)

- `CONTRIBUTING.md`: a new page for whoever maintains this repository, person or agent, in any session: the rules of the text, versioning, the release procedure with the scans and the lint, the decisions already taken, and where decisions and lessons go. The instruction file a cloud session reads points at it.
- `README.md`: the page is listed under the files.

## 1.2.0 (2026-10-21)

- `docs/playbooks/web-seo.md`: a new playbook, search visibility for a web app or site, applied only to what the public reaches through search. The kind of site first; what the first HTML must carry; crawl and index rules (robots, sitemap, canonical, redirects, pagination, languages, crawler decisions by exact name); URLs, security and mobile parity; the three field metrics with thresholds and the image targets; page content (headings, coverage floors, readability, author and dates, trust pages, passages an answer engine can cite); structured data with the types to use and the types never to add; extra checks for local, shop, publisher, product and programmatic sites; the audit, its priorities and the four things every recommendation carries; checks and tests to add.
- `HARNESS.md` section 3, `docs/playbooks.md`, `README.md`: the playbook is named beside the other three, with its scope.

## 1.1.0 (2026-10-21)

- `docs/playbooks/motion-video.md`: a new playbook for motion graphics and video. Storyboard and one hero moment first; one timeline of named beats that scenes, sound and review all read; every frame from the time alone; two pipelines (compositions, and a self-contained page captured frame by frame by a headless browser); motion defaults with numbers; a review loop over contact sheets, with done meaning a clean pass over the latest render; generated sound from the same timeline with level, pan and loudness targets; music-driven pieces; the gallery; checks and tests.
- `HARNESS.md` section 3, `docs/playbooks.md`, `README.md`: the playbook is named beside the other two.
- `README.md`: the parts and the files are lists instead of tables, so they read on a phone.
- `HARNESS.md` section 9, `docs/lessons.md`: memory recall and links. A lesson proved wrong is retired with the date instead of removed, and a changed fact becomes a new entry (an order that changed); one file per cue so a repeat overwrites; bounded recall at session start, two hops at most, nothing cut silently; a local word index when memory outgrows the index file; links that carry their evidence or say they were asserted by hand; nothing invented to fill a gap.

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
