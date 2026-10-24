# Testing

Fast tests that stay accurate: run what a change can reach, run it once, and tell the difference between code that failed and an environment that could not run. The agent builds the project's own test runner from this page.

## Three tiers

1. **Every change** (each slice, and each update of a pull request): the code check on the changed files, plus only the suites the change reaches. A docs-only change (only files the filters `ignore`) runs no suite; content a site publishes is mapped to its suites and kept out of the `ignore` globs, so it is never docs-only. A change to a manifest or lock file a suite lists under `deps`, other than its version field, reaches that suite, and a change to the setup script reaches every suite, so a new dependency or tool is always tested; the version bump alone reaches nothing. A change to the runner or to a check runs everything it gates; a change to the filters file runs only the suites whose own entry or files changed (decision 3, with the exclusions of decision 4). A skipped suite is not a failure. A suite that passed on identical code is not run again.
2. **Before a merge into main** (a deploy): the same filtered run over everything changed since the last release tag (`--since <tag>`), plus the suites marked `expensive` that change selects; a suite marked `uncached` waits for the expensive tier. Packaging and release builds run the full code check gate, each step skipped only on identical input.
3. **The expensive tier**, in place of tier 2 for that merge, before a merge that bumps the first or second number of the version or touches payments or billing, and when the owner asks: every suite (`--all`), the browser engine and device profiles a web app or site owes its users (below) and the persona walkthroughs among them (suites that walk the main flows as each kind of user, cached like any other), the live-like runs, and the full security review of the code (`docs/security-review.md`). It never runs on a calendar: a run over code that did not change proves nothing new. Its full security review takes the place of the scoped one.

The pull request says which tier ran and what was skipped. Nothing needs a hosted CI service: the same runner runs on the developer's machine or in a cloud session. Where the host runs checks as well, one run proves one tree (`docs/shipping.md`, workflow runs on the host).

## Path filters

A filters file maps each suite to what it depends on:

```json
{
  "ignore": ["**/*.md", "docs/**"],
  "baseline": { "from": ["server/db/**", "server/seed/**", "server/package-lock.json"], "tools": ["database"] },
  "suites": {
    "unit-server":   { "paths": ["server/src/**", "server/test/**"], "deps": ["server/package-lock.json"], "tools": ["runtime"] },
    "end-to-end":    { "paths": ["server/src/**", "server/db/**", "server/seed/**", "server/tools/e2e/**", "config/test.json"], "deps": ["server/package-lock.json"], "tools": ["runtime", "database"], "env": ["APP_ENV"] },
    "web-smoke":     { "paths": ["server/src/web/**", "web/**"], "deps": ["server/package-lock.json"], "tools": ["runtime", "browsers"], "baseline": true },
    "web-devices":   { "paths": ["server/src/web/**", "web/**"], "deps": ["server/package-lock.json"], "tools": ["runtime", "browsers"], "baseline": true, "expensive": true },
    "live-like":     { "paths": ["server/src/**"], "deps": ["server/package-lock.json"], "tools": ["runtime"], "expensive": true, "uncached": true },
    "app-unit":      { "paths": ["app/lib/**", "app/test/**"], "full": ["app/pubspec.*", "app/assets/**"], "deps": ["app/pubspec.lock"], "tools": ["app-sdk"] }
  },
  "flaky": [
    { "step": "end-to-end", "since": "2026-01-15", "log": "runs/2026-01-15-1002/end-to-end.log" }
  ]
}
```

Each entry names everything its suite depends on, because what the entry does not name can neither select the suite nor clear its cache:

- `paths`: the files it covers, a file it reads outside its folders (a configuration file, a shared fixture) included.
- `full`: files whose change every test file of the suite must see; each test file then runs once (unit-test selection, below).
- `deps`: the manifests and lock files it depends on. Without the key, every manifest and lock file in a folder that holds one of its paths, or in any folder above it up to the root.
- `tools`: the pinned tools it runs, by their names in the setup script (the runtime, the database engine, the browsers, a mobile SDK). Required: the runner refuses an entry without it, since a suite whose tools are not named cannot be cached safely.
- `env`: the environment variables it reads.
- `baseline`: it loads the test baseline (below).
- `expensive` and `uncached`: as in the tiers above and the exclusions below.

The top-level `baseline` describes that baseline: `from`, every file the baseline step reads (the schema, the migrations, the seed data, and the lock file of the migration tool), and `tools`, what building it needs. The hash of those files, those tools' installed versions and the runner is the baseline's name, known before anything runs.

Decision, in order:

1. `--release` means every suite except the `uncached` ones, plus the packaging steps; the build scripts run it before a release artifact (`docs/code-check.md`, ship step 9). `--all` selects every suite, the ones marked `expensive` and `uncached` included. In both, the pass cache still skips a cached suite that passed on identical input.
2. Otherwise the changed set is the diff against the merge base with main (so a branch's commits count, not only uncommitted work), or against `<ref>` with `--since <ref>`, plus uncommitted and untracked files, minus the `ignore` globs. Before the first release tag, the run before a merge compares against an empty tree, so the whole code base counts. Nothing left means no suite runs.
3. A cached suite is selected when one of its project inputs changed:
   - a changed path matches its `paths` globs (`**` is any depth, `*` stays inside one folder), or its `full` globs, whose change enters every test file's record, so each runs once;
   - a file in its `deps` changed, other than in its version field;
   - the setup script changed, since it installs the tools of every suite;
   - its own entry in the filters file changed, or its hash no longer matches a pass because a file the old `ignore` globs hid now counts;
   - for a suite with `baseline`, the baseline's name changed (a file under the baseline's `from` changed); a change outside its own `paths` that leaves the name as it was does not select it;
   - the runner changed, which selects every cached suite: a change to the gate runs what it gates.

   Machine inputs (the installed versions of its tools, the values of its environment variables) select nothing, since they are not a change to the project and the preflight warns about a drift from the pins; they still clear its cache, so the next run that selects the suite runs it again. An edit to the `flaky` list alone selects nothing.
4. Exclusions, for every selection above: a suite marked `expensive` is left out of `--changed`; it runs with `--since <ref>` (before a merge) when selected, and always with `--all` and `--release`. A suite marked `uncached` runs only with `--all` (the expensive tier) and when the owner asks.
5. The code check step is added whenever a file changed: with `--changed`, the check command in its changed mode; with `--since <ref>`, the same mode against `<ref>`; with `--all` and `--release`, the full gate. Either way the check command skips what came out clean on identical input (`docs/code-check.md`).
6. Order: a suite with `baseline` makes its cache check with the baseline's name, which the files give before anything runs, so `--dry-run` can plan it. When it runs and no baseline is stored under that name, the runner's baseline step makes one first, whatever the targets (database hygiene, below); when that step does not pass, the suite reads `not run (no baseline: the baseline step did not pass)`, and the step itself is FAILED or BLOCKED by its log like any step.

## Browsers and devices for a web app or site

A web system, a web app or a site is used from whatever browser its users already have, so its browser suites prove every screen on all of them. This applies to every project with a web front end; a native app, an API or a command-line tool skips it.

Every current browser runs on one of three engines: the one most desktop browsers and Android share (the default engine below), the one every browser on an iPhone uses, and a third with its own desktop browser. The targets:

- **A Windows desktop** in all three engines. An engine renders the same on any operating system, so one desktop stands in for all of them, and the profile may run on any machine the runner has: Windows is the user agent and the window size, because that is where most desktop users are.
- **Android**, in the default engine.
- **iPhone**, in its own engine.
- **The in-app browsers** of the social and messaging apps the users open links from, each on Android and on iPhone: one profile per app per phone, and the project's instruction file names the apps. They open links with the app's own user agent, a smaller viewport, and without some features the normal browsers have.

How they run:

- The end-to-end tool runs each target as a profile: engine, viewport, touch, pixel ratio and user agent. The profiles live in the test setup, pinned like any other tool, and the preflight checks that the browser binaries are installed.
- Every change that reaches the front end runs two profiles, the desktop in the default engine and Android: they belong to the suite the filters file maps to the front-end paths (`web-smoke` above). The rest (`web-devices` above, marked `expensive`: iPhone, the in-app profiles, and the desktop in the two other engines) run once before a merge whose change reaches the front end, and in the expensive tier, so no front-end change ships untested on any profile and no profile runs on every commit.
- The desktop profile also runs the checks that do not depend on the device: every appearance the project has, an automated contrast check in each, and reduced motion on. The other profiles run once, in the default appearance, with no second contrast check.
- An in-app profile shares its engine with its phone's profile, so it runs only what differs: the layout at its viewport, the safe areas under its own bars, and the fallbacks for the features it lacks. The rest of every screen is proved by the phone's profile.
- A feature the in-app browsers lack is tested by the fallback: the suite stubs the feature as missing in the in-app profiles and asserts the screen's alternative appears.
- A pass on an emulated profile is the normal evidence. A difference that depends on the real operating system or device (fonts, scrollbars, file and date pickers, a keyboard that covers a field, a safe area) is a hand check on the real thing when a change touches it or the owner asks, and its result goes into the pull request text like any other hand check.
- A screen that fails on any profile is FAILED, with the profile named in the step. It is never excused as a known limitation of that browser.

What every screen must pass, each check on the profiles the lines above give it:

- No horizontal scroll at the profile's width, with a 16 pixel side gutter on phones.
- Text fields with a font size of at least 16 pixels, so a phone does not zoom in when a field gets focus.
- The device's safe areas respected: nothing under a notch, a home bar or the in-app browser's own top and bottom bars.
- Touch targets of at least 24 by 24 CSS pixels on the phones; on the desktop, every control reached by the tab key with a visible focus state.
- No feature the in-app browsers lack is used without a fallback: file download, sharing, the clipboard, pop-up windows, notifications, sign-in in a pop-up. The screen says what it cannot do instead of failing silently.
- Text readable on its background in every appearance the project has (`HARNESS.md` section 2): at least 4.5 to 1, and 3 to 1 for large text, icons, field borders and focus outlines, measured once, by the contrast check on the desktop profile, because contrast does not depend on the device. Text over an image, a video or a gradient, which an automated check cannot measure reliably, is judged on the screenshot of the same run.
- No motion when the device asks for reduced motion.

## Isolated suites

Each suite brings its own data and leaves nothing for the next one:

- throw-away accounts created by the suite, never a shared test account;
- a made-up username for the wrong-password check, so it never counts against a real account;
- sign-in and registration rate-limit counters cleared between suites, because back-to-back suites signing in as the same user hit the limit and fail for an environment reason.

## Lanes

A server lane (database, server, API suites, web smokes) and an app lane (unit tests, builds) run in parallel. Steps inside a lane run one after another. A step marked fatal (a build, or the database and server start) that does not pass, FAILED or BLOCKED, ends its lane; every later step in that lane is reported as `not run (an earlier step stopped the lane)`. A packaging step after any step that did not pass is reported as `not run (an earlier step did not pass)`. On a small machine keep the heavy build out of the browser smokes' way: sequential lanes then.

## No rerun on identical code

Each suite gets a hash over: the suite key, the hash of the runner, the suite's own entry in the filters file (not the rest of it: the `ignore` globs change which suites a diff selects, not what a suite's result is, so adding or editing another suite, the `ignore` globs or the `flaky` list reruns nothing by itself), the setup script, the files in its `deps` (so a new dependency is a change, and a lock file it does not use is not), the installed versions of the tools in its `tools` (read from them during the preflight, so a machine that drifted from the pins misses the cache), the names and values of the variables in its `env` (a secret value only as its hash), with the project's version left out in its four places (the manifest's version field, the lock file's version field, the CHANGELOG, and the version part of served asset names and of the references to them), so a renumber at merge reruns nothing, the selected test files, the name of the baseline it loads (so a schema or seed change reaches the suites that read it, and nothing else does), and every file matching its `paths` or `full` globs, tracked or untracked but not ignored (so a new file not yet committed counts, and an asset or a localisation file in `full` is never skipped as unchanged), as `path:sha256`, sorted.

A pass adds the hash to the suite's set in `pass-cache.json`; the set keeps every hash the suite passed on, dropping the oldest only once it holds a few thousand, so switching back to code that passed, a revert included, reruns nothing. When the hash is in the set, the suite is skipped and reported as `skipped (passed on identical code, <date of that pass>)`.

The runner comes with a cache test, as the check command does (`docs/code-check.md`): on a scratch copy it changes each input of a suite's hash in turn and expects a miss when the suite is next selected. The runner reads the installed tool versions, the environment and the clock through one place the test can override, so the inputs that are not files are tested too. It runs when the runner is written and whenever it changes, which already reruns every suite, so it adds no run of its own; a skip the agent suspects is wrong is reported with the pass it reused, and the owner's `--no-cache` run settles it.

Every tier uses the cache, the expensive tier and a `--release` run included: a result on identical code and identical tools is the same result, so running it again proves nothing. The packaging steps of a release always run. A suite whose result depends on something outside the repository (a live-like run against an outside service) is marked `"uncached": true` in the filters file and runs every time it is selected. `--no-cache` forces a rerun when the owner asks for one.

## Unit-test selection by import graph

For unit tests in languages with explicit imports, resolve each test file's closure over its imports, and record each passing test file by the hash of its closure together with every other input of the suite's hash (its `deps`, tools, environment variables, the runner, the setup script, its baseline, the files its `full` globs match), so a dependency update, a new tool or an asset change still runs it, once. A selected suite then runs only the test files whose closure has no pass recorded, so a second slice does not rerun the first slice's tests. A change to a file its `full` globs match (a manifest, a lock file, a localisation file, an asset) therefore misses every record once and runs every test file once, since a closure over imports cannot see that change; later slices on the same branch do not run them again.

## Control time, do not wait

Slow smoke sections usually wait on the application's own timers (polling floors, call timeouts, overlays). Make these waits short in the test setup, and fix stale waits (a check waiting the full timeout on text that no longer exists). Never ship a test-only time scale in production code: it is a hook in shipped code and it turns flaky under load.

## Writing tests

The runner decides what runs; this decides what a test is worth when it does.

- **Most tests small.** A function or a module, in one process, with no network and no database, in milliseconds. Fewer tests cross a boundary (the API with the test database, a component with its children). The end-to-end suite covers the main user flows, not every branch.
- **Real code before stand-ins.** The real implementation first, then an in-memory fake, then a stub that returns fixed data. A mock that checks calls only where the real thing is slow, random or reaches outside (mail, payments, an outside API).
- **Outcomes, not calls.** A test checks what a caller or a user sees, so a refactor that keeps the behaviour keeps the test green.
- **One behaviour per test**, named for that behaviour, laid out as setup, action, check. Some repetition between tests is fine when each one reads on its own.
- **Seen failing once.** A test is seen failing before the code that makes it pass, or on the bug it reproduces (`HARNESS.md` section 3), with the project's test tool on that one test; the runner then shows it passing. A test that has never failed may test nothing.

## Logs and history

- The folder of a run where everything passed is deleted. The folder of a run with a FAILED, BLOCKED or FLAKY step is kept, and the last 25 lines of each such step are printed with the path to the full log.
- At startup, run folders older than 7 days are deleted, except a log that an entry on the known-flaky list points at.
- Every run appends one line to `history.log`: timestamp, mode, verdict (`passed`, `FAILED`, `BLOCKED`, `INCOMPLETE`), seconds, runtime version, then each step with its status and seconds.
- Test output never lives next to the source: it goes to the folder the project names for installs and runs.

## Preflight

Before anything runs, the runner checks what the planned steps need and prints every problem with its fix. A problem found here is the BLOCKED verdict caught before the start, which is why it has its own exit code. Nothing runs when a problem exists:

```text
BLOCKED before the start (nothing ran):
  x runtime 22 or newer is required (found 20.11)  fix: install it from the setup script
  x database engine not found on this machine    fix: run setup/install-db
  x interpreter for the end-to-end suite missing  fix: install it with the database and image extensions
```

Checks: runtime version at or above the minimum; build dependencies installed when a build is planned; the database engine present when server suites are planned, and the baseline's `tools` when a suite marked `baseline` is planned, whatever the targets; the interpreter and extensions the end-to-end suite needs; the mobile SDK when an app step is planned. Tool versions that differ from the versions pinned in the setup script print as warnings (`!`) and do not stop the run. `--dry-run` prints the run plan and the preflight, then exits without running anything.

## Database and server hygiene

- Every run gets its own database, dropped and created again (`<project>_test_run`). The database named in the project's `.env` is never touched.
- A change to the schema or its migrations runs the end-to-end suite (its paths include them).
- A suite marked `baseline` starts from the test baseline, so it has data to work with: the database right after the migrations and the seed data are loaded on a fresh database, dumped by the runner's baseline step. No test writes into it, so its content follows from the files that build it alone. It is stored under its name (the hash of the baseline's `from` files, its tools' installed versions and the runner); a name once stored is never overwritten, and the last five names are kept, since a dump is large. The name changes with those files, those tools or the runner, so five names reach far back, and a revert to an older schema finds its baseline. A change to a file under `from` selects the suites that load it; a new tool version or runner, a machine input, only clears their cache. When no baseline is stored under the current name, the baseline step makes one, whatever the targets; it is a setup step, not a suite, so no suite that passed runs again for it.
- The runner finds the database that belongs to this project by its program path on the known ports and never touches another project's database or server.
- The runner starts the database only if it is down, and stops it afterwards only if this run started it (`--keep` leaves it running). The same for the test server.
- The test server starts on its port or the next free one, with ephemeral or dummy secrets, and has a fixed time (about 90 seconds) to answer its health endpoint.

## Verdicts

Only a step that failed and has a log is classified, from the last 80 lines of that log:

- **BLOCKED**: the log shows an environment cause. A rate limit was hit (sign-in, registration or an API); a network or port error (connection refused, address in use, timeout, name lookup); the local database or server did not start or did not answer its health check; a tool is not installed; an outside service was unavailable. A blocked step is reported apart from failures, with its cause, and is never counted as a code failure. A run that could not happen stays unscored.
- **FAILED**: everything else; the code is treated as wrong.
- **FLAKY**: a step on the known-flaky list (`flaky` in the filters file, each entry with the date and the log of the failure that put it there) that failed with no environment cause in its log runs again at once, once, without a fix. A pass on that rerun makes it FLAKY, a second failure makes it FAILED; an environment cause makes it BLOCKED, as for any step. A FLAKY step does not fail the run, does not end its lane and does not stop a merge, and is never a clean pass: it is listed in the verdict line and in the history, its run folder is kept with the failing log, and the cache stores it as a pass while the list keeps it in view. The list is a queue of steps to fix, not a place to keep them: every pull request text names its entries until each is fixed and removed. A step that is not on the list gets no automatic rerun; it goes on the list only with the log of a failure that passed on identical code, which shows when the same code passes in another run (on another machine, in the run after an environment fix, or with `--no-cache` at the owner's ask). A new entry is named in the pull request text like every entry, and it waits for the owner's go, because it lets a failing step pass on a rerun (`docs/code-review.md`).
- **not run**: a planned step that never started, with the reason. The run plan is what the filters and the options select: a suite they leave out, or one the cache skips, is reported `skipped` with its reason and never makes a run INCOMPLETE.

The verdict line lists FAILED first, then BLOCKED with causes, then not run, then FLAKY, or prints `ALL PASSED`. The run as a whole is `FAILED` when any step failed, `BLOCKED` when none failed but a step was blocked, `INCOMPLETE` when none failed or was blocked but a planned step did not run, and `passed` otherwise, FLAKY steps included. Exit codes: `0` passed (skipped and FLAKY steps listed), `1` a step FAILED, `2` a preflight problem (nothing ran), `3` nothing failed but a step was BLOCKED or did not run. A caller tells the code from the environment by the code alone.

## Options

`[targets]` (server, web, app: only the suites with any of their paths in that part's folders; none means all, and the app build script names the app), `--changed` (default), `--since <ref>`, `--all`, `--release`, `--dry-run`, `--no-cache`, `--keep`. Repeating the same command resumes: suites that passed are skipped by their hash, and the rest run again. It is repeated after a fix or an environment fix, never to retry the same code. `--dry-run` exits `0`, or `2` when the preflight found a problem.

## Debugging a failure

A FAILED step is worked in this order. BLOCKED is not debugged in the code: the environment is fixed and the blocked steps run again. A FLAKY step follows the flaky rule (verdicts, above).

1. **Stop the line.** No new work on top of a failing step: a bug left in one slice makes the next one wrong.
2. **Read the evidence first.** The failing run's folder is kept with its log (logs and history, above); read it before changing anything.
3. **Reproduce** with the project's test tool on the one failing test, against a test database made the way the runner makes one and never the one in the project's `.env`, so each try takes seconds. The runner is not repeated until there is a fix.
4. **Localize**: which part fails (the interface, the server, the database, the build, an outside service, or the test itself). For a regression, bisect between the last good commit and the first bad one with that one test.
5. **Reduce** the case to the smallest input that still fails.
6. **Fix the cause**, not the place it shows, and keep the reproducing test. Once the fix is in, the test tool does not run that test again: the next run is the runner's.
7. **Run the runner.** It shows the reproducing test passing, runs what else the fix reaches and skips what passed on identical code; there is no extra run of the one test, and no full run "to make sure".

When the failure followed an update of a tool or a dependency, the A/B rule below decides first.

## The A/B rule

When an update makes a check fail, change one thing at a time and compare the new result with the old one on the same input, the old one taken from the history rather than from a new run, before deciding. Reruns: a suite that passed on identical code is never run again, not even to "make sure". A failed step runs again after a fix. A step on the known-flaky list runs again once, at once and without a fix, and that rerun is recorded as FLAKY or FAILED. A blocked step runs again once the environment is fixed. The cache holds in every tier; only `--no-cache` and the suites marked `uncached` bypass it.
