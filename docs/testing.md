# Testing

Fast tests that stay accurate: run what a change can reach, run it once, and tell the difference between code that failed and an environment that could not run. The agent builds the project's own test runner from this page.

## Three tiers

1. **Every change** (each pull request): the code check on the changed files, plus only the suites the change reaches. A docs-only change runs no suite. A change to the runner, its filters or a check runs everything it gates. A skipped suite is not a failure. A suite that passed on identical code is not run again.
2. **Before a merge into main** (a deploy): the same filtered run over everything changed since the last release tag (`--since <tag>`). Packaging and release builds keep the full code check gate.
3. **The expensive tier**, weekly or when the owner asks (for example before a big or money-related release): every suite (`--all`), browser engine checks, the device profiles a web app or site owes its users (below), persona walkthroughs and live-like runs.

The pull request says which tier ran and what was skipped. Nothing needs a hosted CI service: the same runner runs on the developer's machine or in a cloud session.

## Path filters

A filters file maps each suite to the paths it covers:

```json
{
  "ignore": ["**/*.md", "docs/**"],
  "suites": {
    "unit-server":   { "paths": ["server/src/**", "server/test/**"] },
    "end-to-end":    { "paths": ["server/src/**", "server/tools/e2e/**"] },
    "web-smoke":     { "paths": ["server/src/web/**", "web/**"] },
    "web-devices":   { "paths": ["server/src/web/**", "web/**"], "expensive": true },
    "app-unit":      { "paths": ["app/lib/**", "app/test/**"], "full": ["app/pubspec.*", "app/assets/**"] }
  }
}
```

Decision, in order:

1. `--release` means `--all` plus the packaging steps. `--all` runs every suite and ignores the pass cache.
2. Otherwise the changed set is the diff against the merge base with main (so a branch's commits count, not only uncommitted work), or against `<ref>` with `--since <ref>`, plus uncommitted and untracked files, minus the `ignore` globs. Nothing left means no suite runs.
3. If the runner or the filters file changed, every suite runs: a change to the gate runs what it gates.
4. Otherwise a suite runs when a changed path matches one of its `paths` globs (`**` is any depth, `*` stays inside one folder). A suite marked `expensive` is left to `--all` and `--release`, whatever changed.
5. The code check step is added whenever a file changed; the check command decides which of its steps the files reach (the text rules for docs, the full set for code).

## Browsers and devices for a web app or site

A web system, a web app or a site is used from whatever browser its users already have, so its browser suites prove every screen on all of them. This applies to every project with a web front end; a native app, an API or a command-line tool skips it.

Every current browser runs on one of three engines: the one most desktop browsers and Android share (the default engine below), the one every browser on an iPhone uses, and a third with its own desktop browser. The targets:

- **A Windows desktop** in all three engines. An engine renders the same on any operating system, so one desktop stands in for all of them, and the profile may run on any machine the runner has: Windows is the user agent and the window size, because that is where most desktop users are.
- **Android**, in the default engine.
- **iPhone**, in its own engine.
- **The in-app browsers** of the social and messaging apps the users open links from, each on Android and on iPhone: one profile per app per phone, and the project's instruction file names the apps. They open links with the app's own user agent, a smaller viewport, and without some features the normal browsers have.

How they run:

- The end-to-end tool runs each target as a profile: engine, viewport, touch, pixel ratio and user agent. The profiles live in the test setup, pinned like any other tool, and the preflight checks that the browser binaries are installed.
- Every change runs two profiles, the desktop in the default engine and Android: they belong to the suite the filters file maps to the front-end paths (`web-smoke` above). The expensive tier runs the rest (`web-devices` above, marked `expensive`): iPhone, the in-app profiles, and the desktop in the two other engines.
- The desktop profile also runs the checks that do not depend on the device: both appearances where the project has them, and reduced motion on. The other profiles run once, in the default appearance.
- A feature the in-app browsers lack is tested by the fallback: the suite stubs the feature as missing in the in-app profiles and asserts the screen's alternative appears.
- A pass on an emulated profile is the normal evidence. A difference that depends on the real operating system or device (fonts, scrollbars, file and date pickers, a keyboard that covers a field, a safe area) is a hand check on the real thing when a change touches it or the owner asks, and its result goes into the pull request text like any other hand check.
- A screen that fails on any profile is FAILED, with the profile named in the step. It is never excused as a known limitation of that browser.

What every screen must pass on every profile:

- No horizontal scroll at the profile's width, with a 16 pixel side gutter on phones.
- Text fields with a font size of at least 16 pixels, so a phone does not zoom in when a field gets focus.
- The device's safe areas respected: nothing under a notch, a home bar or the in-app browser's own top and bottom bars.
- Touch targets of at least 24 by 24 CSS pixels on the phones; on the desktop, every control reached by the tab key with a visible focus state.
- No feature the in-app browsers lack is used without a fallback: file download, sharing, the clipboard, pop-up windows, notifications, sign-in in a pop-up. The screen says what it cannot do instead of failing silently.
- Readable in both appearances where the project has them, and no motion when the device asks for reduced motion.

## Isolated suites

Each suite brings its own data and leaves nothing for the next one:

- throw-away accounts created by the suite, never a shared test account;
- a made-up username for the wrong-password check, so it never counts against a real account;
- sign-in and registration rate-limit counters cleared between suites, because back-to-back suites signing in as the same user hit the limit and fail for an environment reason.

## Lanes

A server lane (database, server, API suites, web smokes) and an app lane (unit tests, builds) run in parallel. Steps inside a lane run one after another. A step marked fatal (a build, or the database and server start) ends its lane on failure; every later step in that lane is reported as `not run (an earlier step failed)`. A packaging step after any failure is reported as `not run (a test failed)`. On a small machine keep the heavy build out of the browser smokes' way: sequential lanes then.

## No rerun on identical code

Each suite gets a hash over: the suite key, the hash of the runner and the filters file, the selected test files, and every tracked file matching its paths as `path:sha256`, sorted. A pass stores the hash in `pass-cache.json`. When the hash equals the stored one, the suite is skipped and reported as `skipped (passed on identical code)`. `--no-cache` forces a rerun. A `--release` run (packaged artifacts) always runs everything; a merge into main runs tier 2 with the cache.

## Unit-test selection by import graph

For unit tests in languages with explicit imports, resolve each test file's closure over its imports and run only the tests that reach a changed file. When 40 percent or more of the tests are selected, or when a manifest, lock file, localisation file or asset changed (`full` globs), run the whole suite.

## Control time, do not wait

Slow smoke sections usually wait on the application's own timers (polling floors, call timeouts, overlays). Make these waits short in the test setup, and fix stale waits (a check waiting the full timeout on text that no longer exists). Never ship a test-only time scale in production code: it is a hook in shipped code and it turns flaky under load.

## Logs and history

- A passing run's folder is deleted. A failing run's folder is kept, and the last 25 lines of each failed or blocked step are printed with the path to the full log.
- At startup, run folders older than 7 days are deleted.
- Every run appends one line to `history.log`: timestamp, mode, verdict (`passed`, `FAILED`, `BLOCKED`, `INCOMPLETE`), seconds, runtime version, then each step with its status and seconds.
- Test output never lives next to the source: it goes to the folder the project names for installs and runs.

## Preflight

Before anything runs, the runner checks what the planned steps need and prints every problem with its fix. Nothing runs when a problem exists:

```text
BLOCKED before the start (nothing ran):
  x Node 22 or newer is required (found 20.11)    fix: install Node 22 from the setup script
  x database engine not found on this machine    fix: run setup/install-db
  x interpreter for the end-to-end suite missing  fix: install it with the database and image extensions
```

Checks: runtime version at or above the minimum; build dependencies installed when a build is planned; the database engine present when server suites are planned; the interpreter and extensions the end-to-end suite needs; the mobile SDK when an app step is planned. Tool versions that differ from the versions pinned in the setup script print as warnings (`!`) and do not stop the run. `--dry-run` prints the plan and the preflight, then exits without running anything.

## Database and server hygiene

- Every run gets its own database, dropped and created again (`<project>_test_run`). The database named in the project's `.env` is never touched.
- When the end-to-end suite does not run, the run database is loaded from the baseline dump the last passing end-to-end run left on a fresh database, so the smokes have data to work with. A passing end-to-end run on a fresh database saves a new baseline.
- The runner finds the database that belongs to this project by its program path on the known ports and never touches another project's database or server.
- The runner starts the database only if it is down, and stops it afterwards only if this run started it (`--keep` leaves it running). The same for the test server.
- The test server starts on its port or the next free one, with ephemeral or dummy secrets, and has a fixed time (about 90 seconds) to answer its health endpoint.
- Between suites, sign-in and registration rate-limit counters are cleared.

## Verdicts

Only a step that failed and has a log is classified, from the last 80 lines of that log:

- **BLOCKED**: the log shows an environment cause. A rate limit was hit (sign-in, registration or an API); a network or port error (connection refused, address in use, timeout, name lookup); the local database or server did not start or did not answer its health check; a tool is not installed; an outside service was unavailable. A blocked step is reported apart from failures, with its cause, and is never counted as a code failure. A run that could not happen stays unscored.
- **FAILED**: everything else; the code is treated as wrong. A step whose name is on the known-flaky list is reported FAILED with the reason `a check known to fail now and then: rerun once to confirm`, runs again once without a fix, and then its result stands.
- **not run**: a planned step that never started, with the reason.

The verdict line lists FAILED first, then BLOCKED with causes, then not run, or prints `ALL PASSED`. The run as a whole is `FAILED` when any step failed, `BLOCKED` when none failed but a step was blocked, `INCOMPLETE` when none failed or was blocked but a planned step did not run, and `passed` otherwise. Exit codes: `0` clean, `1` anything failed, was blocked or did not run, `2` a preflight problem.

## Flags

`--changed` (default), `--since <ref>`, `--all`, `--release`, `--dry-run`, `--no-cache`, `--keep`. Repeating the same command resumes: suites that passed are skipped by their hash, and the rest run again.

## The A/B rule

When an update makes a check fail, change one thing at a time and compare old and new on the same input before deciding. Reruns: a suite that passed on identical code is never run again, not even to "make sure". A failed step runs again after a fix. A known flaky step runs again once without a fix, and that rerun is recorded. A blocked step runs again once the environment is fixed. `--all` and `--release` ignore the cache by design.
