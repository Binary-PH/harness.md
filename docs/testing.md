# Testing

Fast tests that stay accurate: run what a change can reach, run it once, and tell the difference between code that failed and an environment that could not run. The agent builds the project's own test runner from this page, on top of what the project's test tools already do.

## Three tiers

1. **Every change** (each slice): only the suites the change reaches, not the suites marked `expensive` or `uncached`. A change only to files the filters `ignore` runs no suite; content a site publishes is mapped to its suites and kept out of the `ignore` globs, so it is never docs-only. A skipped suite is not a failure.
2. **Before a merge into main** (a deploy): the suites that everything changed since the last release tag reaches (`--since <tag>`; before the first tag, every suite), plus the suites marked `expensive` that it reaches. A suite marked `uncached` waits for the expensive tier.
3. **The expensive tier**, in place of tier 2 for that merge: every suite (`--all`), the `uncached` ones included, and the full security review of the code (`docs/security-review.md`). It runs before a merge that bumps the first or second number of the version or touches payments or billing, and when the owner asks; never on a calendar, since a run over code that did not change proves nothing new.

The pull request says which tier ran and what was skipped. Nothing needs a hosted CI service: the same runner runs on the developer's machine or in a cloud session. Where the host runs checks as well, one run proves one tree (`docs/shipping.md`, workflow runs on the host).

## The filters file

A filters file maps each suite to the paths it covers. A project with one suite needs no `suites` map: its one suite covers every tracked file the `ignore` list leaves in, and the file holds only `ignore` and `flaky`.

```json
{
  "ignore": ["**/*.md", "docs/**"],
  "suites": {
    "unit-server":   { "paths": ["server/src/**", "server/test/**"] },
    "end-to-end":    { "paths": ["server/src/**", "server/db/**", "server/tools/e2e/**"] },
    "web-smoke":     { "paths": ["server/src/web/**", "web/**"] },
    "web-devices":   { "paths": ["server/src/web/**", "web/**"], "expensive": true },
    "live-like":     { "paths": ["server/src/**"], "expensive": true, "uncached": true },
    "app-unit":      { "paths": ["app/lib/**", "app/test/**"], "full": ["app/pubspec.*", "app/assets/**"] }
  },
  "flaky": [
    { "step": "end-to-end", "since": "2026-01-15", "log": "runs/2026-01-15-1002/end-to-end.log" }
  ]
}
```

A file a suite reads outside its folders (a configuration file, a shared fixture) goes into its `paths`, and a file a suite's `paths` match is never ignored for that suite. A suite's folders are the top folders of its `paths`. A suite is selected when:

- a changed path matches its `paths` globs (`**` is any depth, `*` stays inside one folder), or its `full` globs, whose change runs all its tests rather than the affected ones;
- a manifest or a lock file in one of its folders changed, other than in its version field, or the lock file at the root of the repository when its folders hold no lock file of their own;
- the setup script or the runner changed, which selects every suite: a change to the gate runs what it gates;
- its own entry in the filters file changed.

The changed set is the diff against the merge base with main (`--changed`, the default) or against `<ref>` (`--since <ref>`), plus uncommitted and untracked files, minus the files the `ignore` globs match and no suite's `paths` match. A suite marked `expensive` is left out of `--changed` and runs with `--since` and `--all`; a suite marked `uncached` runs only with `--all` and when the owner asks. `--release`, which the build scripts run before a release artifact (`docs/code-check.md`), means every suite but the `uncached` ones; the packaging steps belong to the build script. A target names a part (server, web, app) or a suite.

## No result twice

A result on identical code and identical tools is the same result, so a suite that passed is not run again on the same input, in any tier.

- **A pass record per suite decides whether it runs.** The runner hashes the suite key, the runner, the suite's own entry in the filters file, the setup script, the manifests and lock files of the suite's folders (and the root's lock file when its folders hold none), and every file its `paths` and `full` globs match, untracked ones included, as `path:sha256`, sorted. A pass, a FLAKY one included, adds the hash to the suite's set in `pass-cache.json`, kept in the folder the project names for installs and runs; the set keeps every hash the suite passed on, so returning to code that passed, a revert included, reruns nothing. The project's version is left out wherever it appears: the version fields of the manifest and the lock file, and the version part of served asset names and of the references to them, so a renumber reruns nothing. A suite whose hash is in the set reads `skipped (passed on identical code)`. A fresh clone has no record and runs what it needs once.
- **The test tool narrows inside.** Inside a suite that runs, the test tool's own result cache or its affected-test mode (the result cache of `go test`, a changed-files mode) may narrow which tests run; a `full` match runs them all.
- **Drift.** When the preflight finds a tool that drifted from its pin, the run neither uses nor writes pass records, since its results may differ.
- **The bypasses.** A suite whose result depends on something outside the repository (a live-like run against an outside service) is marked `uncached` and runs every time it is selected. `--no-cache` forces a run when the owner asks. The packaging steps of a release always run.

Keep the record this plain. A cache that tries to track every input by hand is where wrong skips come from. When a skip looks wrong, the owner's `--no-cache` run settles it; when that run fails, the input the hash missed goes into the suite's `paths`.

## Browsers and devices for a web app or site

A web system, a web app or a site is used from whatever browser its users already have, so its browser suites prove every screen on all of them. This applies to every project with a web front end; a native app, an API or a command-line tool skips it.

Every current browser runs on one of three engines: the one most desktop browsers and Android share (the default engine below), the one every browser on an iPhone uses, and a third with its own desktop browser. The targets:

- **A Windows desktop** in all three engines. An engine renders the same on any operating system, so one desktop stands in for all of them, and the profile may run on any machine the runner has: Windows is the user agent and the window size, because that is where most desktop users are.
- **Android**, in the default engine.
- **iPhone**, in its own engine.
- **The in-app browsers** of the social and messaging apps the users open links from, each on Android and on iPhone: one profile per app per phone, and the project's instruction file names the apps. They open links with the app's own user agent, a smaller viewport, and without some features the normal browsers have.

How they run:

- The end-to-end tool runs each target as a profile: engine, viewport, touch, pixel ratio and user agent. The profiles live in the test setup, pinned like any other tool, and the preflight checks that the browser binaries are installed.
- Every change that reaches the front end runs two profiles, the desktop in the default engine and Android (`web-smoke` above). The rest (`web-devices` above, marked `expensive`: iPhone, the in-app profiles, and the desktop in the two other engines) run once before a merge whose change reaches the front end, and in the expensive tier.
- The desktop profile also runs the checks that do not depend on the device: every appearance the project has, an automated contrast check in each, and reduced motion on. The other profiles run in the default appearance, and an in-app profile runs only what differs from its phone's profile: the layout at its viewport, the safe areas under its bars, and the fallbacks for the features it lacks.
- A feature the in-app browsers lack is tested by the fallback: the suite stubs the feature as missing and asserts the screen's alternative appears.
- A pass on an emulated profile is the normal evidence. A difference that depends on the real device (fonts, scrollbars, file and date pickers, a keyboard that covers a field, a safe area) is a hand check on the real thing when a change touches it or the owner asks, reported in the pull request text.
- A screen that fails on any profile is FAILED, with the profile named in the step, never excused as a known limitation of that browser.

What every screen must pass, each check on the profiles above:

- No horizontal scroll at the profile's width, with a 16 pixel side gutter on phones.
- Text fields with a font size of at least 16 pixels, so a phone does not zoom in when a field gets focus.
- The device's safe areas respected: nothing under a notch, a home bar or the in-app browser's own bars.
- Touch targets of at least 24 by 24 CSS pixels on the phones; on the desktop, every control reached by the tab key with a visible focus state.
- No feature the in-app browsers lack is used without a fallback (file download, sharing, the clipboard, pop-up windows, notifications, sign-in in a pop-up); the screen says what it cannot do instead of failing silently.
- Text readable on its background in every appearance the project has (`HARNESS.md` section 2): at least 4.5 to 1, and 3 to 1 for large text, icons, field borders and focus outlines, measured on the desktop profile; text over an image, a video or a gradient is judged on the screenshot of the same run.
- No motion when the device asks for reduced motion.

## Writing tests

The runner decides what runs; this decides what a test is worth when it does.

- **Most tests small.** A function or a module, in one process, with no network and no database, in milliseconds. Fewer tests cross a boundary. The end-to-end suite covers the main user flows, not every branch.
- **Real code before stand-ins.** The real implementation first, then an in-memory fake, then a stub that returns fixed data. A mock that checks calls only where the real thing is slow, random or reaches outside (mail, payments, an outside API).
- **Outcomes, not calls.** A test checks what a caller or a user sees, so a refactor that keeps the behaviour keeps the test green.
- **One behaviour per test**, named for that behaviour, laid out as setup, action, check.
- **Seen failing once.** A test is seen failing before the code that makes it pass, or on the bug it reproduces (`HARNESS.md` section 3); the runner then shows it passing. A test that has never failed may test nothing.

## Isolated suites

Each suite brings its own data and leaves nothing for the next one: throw-away accounts created by the suite, never a shared test account; a made-up username for the wrong-password check; sign-in and registration rate-limit counters cleared between suites, because back-to-back suites signing in as the same user hit the limit and fail for an environment reason.

## Database and server hygiene

When the project has a database or a server:

- Every run gets its own database, dropped and created again from the migrations and the seed data (`<project>_test_run`). The database the project itself uses is never touched, and neither is another project's database or server.
- A project whose seed is slow may keep a dump of the freshly seeded database, named by the hash of the files that build it (the schema, the migrations, the seed data) and the setup script, and load it instead of seeding again; a change to any of them makes a new dump, and only the newest is kept.
- The runner starts the database only if it is down, and stops it afterwards only if this run started it (`--keep` leaves it running). The same for the test server, which starts on its port or the next free one, with dummy secrets, and has a fixed time (about 90 seconds) to answer its health endpoint.

## Lanes

When the project has more than one part (a server and an app), their suites run in two lanes in parallel; steps inside a lane run one after another. A fatal step (a build, or the database and server start) that does not pass ends its lane, and every later step in that lane reads `not run (an earlier step stopped the lane)`. On a small machine, the lanes run one after the other.

## Control time, do not wait

Slow tests usually wait on the application's own timers (polling floors, call timeouts, overlays). Make these waits short in the test setup, and fix stale waits. Never ship a test-only time scale in production code.

## Preflight

Before anything runs, the runner checks what the planned steps need and prints every problem with its fix. A problem found here is the BLOCKED verdict caught before the start, which is why it has its own exit code. Nothing runs when a problem exists:

```text
BLOCKED before the start (nothing ran):
  x runtime 22 or newer is required (found 20.11)  fix: install it from the setup script
  x database engine not found on this machine    fix: run setup/install-db
```

Checks: the runtime version; build dependencies when a build is planned; the database engine when server suites are planned; the interpreters, browsers and SDKs the planned suites need. A tool version that differs from its pin is a warning (`!`). `--dry-run` prints the run plan and the preflight, then exits.

## Verdicts

Only a step that failed and has a log is classified, from the last lines of that log:

- **BLOCKED**: the log shows an environment cause: a rate limit, a network or port error, a database or server that did not start, a missing tool, an outside service down. Reported apart from failures, with its cause, never counted as a code failure. A blocked step runs again once the environment is fixed.
- **FAILED**: everything else; the code is treated as wrong. A failed step runs again after a fix.
- **FLAKY**: a step on the known-flaky list (`flaky` in the filters file, each entry with the date and the log of a failure on code that passed elsewhere: on another machine, or in a rerun the owner approved) that fails with no environment cause runs again at once, once, without a fix: a pass makes it FLAKY, a second failure FAILED. A FLAKY step does not fail the run and does not stop a merge, but it is never a clean pass: it is listed in the verdict line and the history, and every pull request text names the list's entries until each is fixed. A new entry waits for the owner's go, because it lets a failing step pass on a rerun.
- **not run**: a planned step that never started, with the reason. A suite the filters leave out, or the pass record skips, is `skipped`, never not run.

The run as a whole is `FAILED` when any step failed, `BLOCKED` when none failed but a step was blocked, `INCOMPLETE` when none failed or was blocked but a planned step did not run, and `passed` otherwise. Exit codes: `0` passed (skipped and FLAKY steps listed), `1` a step FAILED, `2` a preflight problem (nothing ran), `3` nothing failed but a step was BLOCKED or did not run. A caller tells the code from the environment by the code alone.

## Logs and history

- A run where everything passed leaves no folder. A run with a FAILED, BLOCKED or FLAKY step keeps its folder, and the last 25 lines of each such step are printed with the path to the full log.
- Run folders older than 7 days are deleted at startup, except a log a known-flaky entry points at.
- Every run appends one line to `history.log`: time, mode, verdict, seconds, then each step with its status.
- Test output never lives next to the source: it goes to the folder the project names for installs and runs.

## Options

`[targets]` (a part, server, web or app, meaning the suites with a path in its folders, or a suite's name; none means all), `--changed` (default), `--since <ref>`, `--all`, `--release`, `--dry-run`, `--no-cache`, `--keep`. Repeating the same command resumes: what passed is skipped, the rest runs. It is repeated after a fix or an environment fix, never to retry the same code.

## Debugging a failure

A FAILED step is worked in this order. A BLOCKED step is not debugged in the code: the environment is fixed and the blocked steps run again.

1. **Stop the line.** No new work on top of a failing step.
2. **Read the evidence first**: the kept log, before changing anything.
3. **Reproduce** with the project's test tool on the one failing test, against a test database, never the project's own, so each try takes seconds.
4. **Localize**: which part fails. For a regression, bisect between the last good commit and the first bad one with that one test.
5. **Reduce** the case to the smallest input that still fails.
6. **Fix the cause**, not the place it shows, and keep the reproducing test.
7. **Run the runner.** It shows the fix passing and runs what else the fix reaches; there is no extra run "to make sure".

When the failure followed an update of a tool or a dependency, the A/B rule decides first: change one thing at a time and compare the new result with the old one on the same input, the old one taken from the history, not from a new run.
