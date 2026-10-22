# Changelog

## 1.8.0 (2026-10-23)

- `HARNESS.md` section 1: a new order, a project's copy of the harness stays at the latest release. In the first report of each session the agent reads the release tags with git, compares the newest with the version in the title, and when it is newer replaces `HARNESS.md` and its pages in `docs/` from a depth-1 clone of that tag (removing the pages the release dropped, never a file the harness did not bring), reads the new file, and reports the old and new version with the CHANGELOG entries between them. The replacement ships as its own pull request and bumps the project's third number, as a dependency update does, and a release that changes the gate or the runner brings the project's check command and runner in line in a change of their own. No go is needed, because everyone follows the latest release. Without git or a network it reports `update not checked`. Reading the file from its address needs no check. Section 4 and `docs/tools-by-language.md`: the harness's own release joins the downloads allowed.
- `HARNESS.md` section 5 and `docs/testing.md`: the expensive tier runs on events, not weekly: before a merge that bumps the first or second number of the version or touches payments or billing, and when the owner asks; never on a calendar, because a run over code that did not change proves nothing new. The cost: a long run of third-number releases leaves the device profiles unrun until a larger release or the owner's ask.
- `HARNESS.md` section 7 and `docs/shipping.md`: step 3 names the expensive tier among the checks when its trigger holds, and step 7 runs the checks of the deployed surface once a release that tier gated is live.
- `HARNESS.md` section 6, `docs/security-review.md`, `docs/security-posture.md`, `docs/playbooks/web-seo.md`: the full security review of the code runs with the expensive tier or when the owner asks; the periodic review is now the review of the deployed surface, run once a release the expensive tier gated is live and after a change to DNS, hosting, certificates or mail settings.
- `docs/security.md` is renamed `docs/security-posture.md`, so its name says what its title says and is not read as the general security page beside `docs/security-review.md`. Every reference follows.
- `README.md`, `CONTRIBUTING.md`: the copy, first use and decisions already taken say the same; the release procedure adds that a release is not finished until its tag is pushed, and that a missing tag is created on its merge commit before the next change.

## 1.7.0 (2026-10-22)

- `HARNESS.md` section 1: a new order, the project's instruction file points at the harness by its address, or at the project's copy, and writes no harness version. The version followed is the one in the title of the file read; a number in the instruction file goes stale with the next release and can be read as a pin, so an agent skips the orders added since.
- `README.md` (first use) and `CONTRIBUTING.md` (decisions already taken) say the same.

## 1.6.0 (2026-10-22)

- `docs/security-review.md`: a new page, the security review of the code: what a reader finds and a tool does not. When it runs (scoped to the diff before a pull request that touches a trust boundary; full when the owner asks, before a big or money-related release, and with the expensive tier). Its rules: source and a sandbox only, never the deployed system; what the repository does not show is unknown, not assumed; a finding is checked by a reader who did not hunt it; findings are records and the report is derived from them; no padding. The six things a finding names, each with file and line, and the smallest fix with a regression test. Three verdicts kept apart: confirmed (the only one with a severity), needs validation (one decisive fact outside the repository, with the owner's safe check), rejected (kept). Severity anchors from critical to informational, capped by the observed impact. A coverage list of entry surface by boundary by class as the claim, with a gap pass by a fresh reader and the next review starting from the last list. The classes of flaw to read for, and the classes per kind of project. The report's parts.
- `HARNESS.md` section 6: a new order, the review summarised, with the section now covering the tools, the review and the deployed surface. Section 5: the full review is named in the expensive tier. Section 7: the scoped review is a check of step 3 and the pull request text carries it when one ran.
- `docs/testing.md`, `docs/shipping.md`, `docs/security.md`, `README.md`, `CONTRIBUTING.md`: the tier, ship step 3 and the pull request text, the split between gate, review and periodic review, the summary lines and the decisions already taken say the same.

## 1.5.0 (2026-10-22)

- `HARNESS.md` section 5: a new order, a web system, web app or site is tested on every browser its users have: a Windows desktop in every browser engine, Android, iPhone, and the in-app browsers of the social and messaging apps users open links from, on both phones, so every screen works in any browser on any operating system. The desktop profile in the default engine and the Android profile run on every change; iPhone, the in-app profiles and the other desktop engines run in the expensive tier. A failure on one profile is FAILED, never a known limitation.
- `docs/testing.md`: a new section with the detail: the three engines and the profiles, why one desktop stands in for every desktop operating system, how each target runs as a pinned profile in the end-to-end tool (engine, viewport, touch, pixel ratio, user agent), the split across the tiers with a suite marked `expensive` in the filters file (a decision rule says such a suite is left to `--all` and `--release`), the fallback test for features the in-app browsers lack, when a difference is a hand check on a real device, and what every screen must pass on every profile (no horizontal scroll, 16 pixel fields, safe areas, touch targets, fallbacks, both appearances, reduced motion).
- `README.md`, `CONTRIBUTING.md`: the summary lines and the decisions already taken say the same.

## 1.4.0 (2026-10-21)

- `HARNESS.md` section 2: a new working rule, keys and passwords are hidden on screen. Where the product's user interface shows a saved key, token or other secret, takes one in, or takes a password, the field hides the value by default and a toggle beside it shows it and hides it again; the toggle works by keyboard and has a name a screen reader reads; every open of the view starts hidden and the shown state is never saved; a copy control may copy without showing.
- `docs/security.md`: the defensive design section carries the detail: where the rule applies (settings pages, forms, detail views, lists with one toggle per row, a field for a new key, every password field), the toggle's names, the shown state never saved, no secret in the page title, the URL or a log, a password field marked as a password.
- `CONTRIBUTING.md`: the decision is listed among the decisions already taken.

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
