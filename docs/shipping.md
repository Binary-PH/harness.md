# Shipping

The main branch is production: the host builds and installs every merge into it. Only checked work goes there, and every deploy carries a version bump. A part of a feature that users must not see yet goes there behind a feature flag that is off (changes a running version must survive, below); it still bumps the version and deploys like any merge.

## The repository and its remote

Every step below assumes a git repository with a remote the owner controls. A project that is not yet in git goes into git before the first feature.

- **No repository yet.** `git init`, a `.gitignore` for the stack, a `.gitattributes` with `* text=auto eol=lf`, one first commit. Then the owner creates the remote, or gives you the access to create it; push main and set it as the upstream. That setup commit is the only one that goes straight to main and carries no version bump; everything after it follows the ten steps. An existing repository without a remote gets only the remote and the push; its uncommitted work goes through a branch.
- **Choosing a host.** A host qualifies when its free plan gives private repositories, pull requests with squash merge through an API, a credential that can be limited to one repository (a scoped token or a deploy key), and a way to take the repository, issues and pull requests elsewhere. Hosted services with a free plan meet this for most projects. A self-hosted server such as Gitea runs as one container and fits an owner who already runs a server with a public hostname and TLS; a server on a home machine is not reachable from a cloud sandbox. Name two or three options with the reason and let the owner pick.
- **Windows.** Git is not preinstalled. Install it with the system package manager or the official installer, open a new terminal and confirm with `git --version`. Set `core.autocrlf` to `false` and let `.gitattributes` decide line endings, so diffs and the text rules are the same on every machine.
- **A session on a cloud sandbox.** The sandbox is thrown away when the session ends. Clone at the start, commit small, push the branch at every checkpoint and before the session ends. Reach the remote through the tool's own repository integration where one exists; otherwise through a credential limited to that one repository (a scoped token or a deploy key) with the shortest life the host allows, stored in the tool's secret settings, never pasted in the chat.

## The ten steps

1. **Branch.** Work on `<agent>/<topic>`, never on main. Start from an up-to-date main (`git switch main && git pull --ff-only`). One task, one branch, one pull request: prompts that continue the task add to its branch, a new task starts a branch of its own, and steps 2 to 10 run once for the task, when its work is done or the owner asks to ship, apart from the changed-files gate before each commit; a commit added after they ran follows `HARNESS.md` section 1 (what runs when). Tasks stay small and merge soon; a branch that lives long drifts from main and grows into a pull request no reader can hold.
2. **Bump the version.** A small change (a fix, wording, a dependency update) bumps the third number. A mid-size change (a feature, a new rule or page) bumps the second. A large change (a rewrite, a break in what users rely on) bumps the first. The numbers after the bumped one reset to 0: from 1.0.0 that is 1.0.1, 1.1.0 or 2.0.0. Count from the version on main when the branch merges: a branch that fell behind updates from main and takes the next free number. Update the manifest, the lock file where it carries the version, and the CHANGELOG. If an app changed, bump its version as well, in the same task. Static assets the project serves get the new version, or a content hash, in their file names (a content hash changes on its own), with every reference updated, so no cache serves a stale copy; a query string on an unchanged name is not enough.
3. **Run the checks once.** The changed-files code check before each commit (`check --changed --gate`). Before the merge, the reviews come first, so a fix they bring is part of the change and the runs that follow check it once: first the security review (`docs/security-review.md`), scoped when the change touched a trust boundary and full whenever the expensive tier runs, in place of the scoped one; then the review of the diff (`docs/code-review.md`), which so reads the security fixes too, with a second reader when the change is hard to undo. Then the dependency step over every lock file (`check --deps --gate`), then tier 2, the filtered suites since the last release tag with the `expensive` suites they reach (`test --since <tag>`, `docs/testing.md`; no tag yet means everything), or the expensive tier (`test --all`) in its place when the version bumps the first or second number, the change touches payments or billing, or the owner asks. BLOCKED means fix the environment and run the blocked steps again with the runner; a workflow run on the host follows the rule below. Never rerun a suite that passed on identical code.
4. **Save the lessons and refresh the memory snapshot**: the task's lessons go into memory now (`docs/lessons.md`), and the snapshot in the repository is refreshed if the project keeps one (`docs/context/memory/`), so the merge carries what the session learned.
5. **Open a pull request** that says what changed, why, the plan or where it lives (`docs/planning.md`), which test tier ran and what was skipped, the FLAKY steps and the entries on the known-flaky list, any step merged BLOCKED as not proved (step 6), the review of the diff with any check it found made easier to pass, when the change touched a trust boundary which boundaries the security review read and what it found (`docs/security-review.md`), any feature flag with its removal date, the way back, and the version change: the template below. No co-author line and no "generated with" line.
6. **Squash-merge** through the platform's API with the title `<title> (#<number>)`, only when the checks pass and the owner approved. A step BLOCKED by a cause outside the agent's reach (an outside service down) does not pass; the owner decides between waiting and merging with that step named in the pull request text as not proved. Read the result: delete the branch only when the response says the merge happened. If the merge is refused, say so and stop.
7. **Verify.** Poll the production health endpoint until it shows the new version (usually one to two minutes). A project without one adds an endpoint, or a version file the host serves, that reports the version; a project that publishes a package (a library, a command-line tool) checks the version the registry shows. If it does not, read the deploy log before anything else. A failed build leaves the old version running; say which version is live. When the new version is live but a main flow is broken, take the way back first (below), then debug. When the expensive tier ran for this release, the checks of the deployed surface follow now, at the levels the owner allowed (`docs/security-posture.md`).
8. **Tag** the merge commit `v<version>` (annotated) and push the tag, once the new version is live. A version that never went live is not tagged.
9. **Release artifacts** (installers, app packages) go through the gated build script only, never a hand-run build: the full gate, then the runner's release run (`test --release`), whose cache skips every suite that passed on identical code (`docs/code-check.md`).
10. **Clean up.** No test server or database you started left running, one-off files deleted.

## Pull request text

```text
What changed: <one paragraph>
Why: <the request or the problem>
Plan: <goal, success criteria, slices> | docs/context/plans/<topic>.md
Tests: tier <2|3>; ran <suites>; skipped <suites and why>; code check: clean | <n> reviewed false positives
Flaky: FLAKY steps none | <steps>; known-flaky list: empty | <entries>
Blocked: none | <step, cause, not proved, owner chose to merge>
Flags: none | <flag, off by default, removed by <date>>
Review: <n> findings fixed; trade-offs <...>; loosened: none | <check, why, owner's go>; second reader: not needed | <n> rounds
Security review: none, no trust boundary touched | not needed, a revert to a reviewed release | scoped: <boundaries read>; <n> confirmed, <n> need validation | full, with the expensive tier: <n> confirmed, <n> need validation, coverage <rows covered of rows>
Way back: revert the merge | <the steps, for a change that is hard to undo>
Version: <old> to <new>
```

## Changes a running version must survive

Main is production, so every merge meets the data, the clients and the users of the version before it.

- **A schema change: expand, then contract.** A change the running code would not survive is spread over releases: add the new column or table, write to both, copy the old data across, move the reads, and only in a later release drop the old one. Never rename or drop in the release whose code stops using it. Each migration carries its own way back.
- **An interface other software reads** (an API, a file format, a message, a public function): add, do not change. A field, a route or an option is removed only after the CHANGELOG has announced it as deprecated and its users have moved; the removal is a break in what users rely on and bumps the first number.
- **A feature flag for a part users must not see yet.** It is off by default, read from configuration kept in the repository rather than buried in the code, and named in the pull request with the date by which it is removed. Its configuration file is in the paths of the suites for the screens it gates, and those suites set it on for the tests of the flagged part, so the flagged code is tested while it is off and turning it on runs them again. A slice that merges behind it changes nothing users see and bumps the third number. Turning it on is the change users see: a pull request through the ten steps that bumps the second number, so the expensive tier runs before it. Once the feature is on for everyone, the flag and the old path are removed in a change of their own. No flag inside another flag.
- **The way back.** A plan for a change that is hard to undo (`HARNESS.md` section 3) says how the release is undone, and the pull request text repeats it. When a live release breaks a main flow, the way back comes first and the debugging after it. For most changes the way back is a revert of the merge through the ten steps, with its own version bump (the third number) and the owner's approval; it brings back code that already passed, so it takes the tier its version and content call for like any merge, and the pass cache, which keeps the hashes of the last release tags, skips every suite that passed on that code. A revert whose tree equals a release tag, apart from the version in its four places, needs no new scoped security review: that tree was reviewed, or needed none, when it shipped. When its content still calls for the expensive tier (it touches payments or billing), the tier runs as always, and its full security review starts from the last full review's coverage list and the scoped reviews since, which together covered that tree when it shipped. For a change that is hard to undo, a revert does not restore what was there, so the way back is the one its plan wrote; data is brought back only by the migration's own way back, never by hand on the production database. Where neither restores the flow, the owner decides between a fix forward and a restore.

## Commit authorship

Commits carry the project's author only. No co-author trailer, no tool-generated trailer, no link to a session. The author name is the repository's configured `user.name`.

## Dependency update bot pull requests

The bot opens a grouped pull request on a schedule. Handle it once:

1. Check out the pull request, rebase it onto main, push with `--force-with-lease`.
2. Install (a plain install, not a clean install from the lock file, so the lock updates).
3. For each major update read its release notes and answer three questions: does our code use the API that changed (search for it)? Did a default we rely on change? Does the package now ship its own types (then remove the separate types package)?
4. Test as for any change (`docs/testing.md`); suites that passed on identical code are skipped. Several updates can share one pull request.
5. If a check fails: change one thing at a time and compare old against new on the same input.
6. Keep a held-back list with the reason per package, mirrored in the bot's configuration, and close those pull requests with a comment.
7. The version bump and the checks apply as for any change.

## Workflow runs on the host

The host runs its own workflows for a push of a working branch, for a pull request, for a merge into main and for the release tag of step 8; those runs are the ship flow and need no extra go. Everything else waits for the owner's word: a run started by hand, a rerun of a failed run on the host, a workflow file added or changed, and a push (any other tag, or a branch with a trigger of its own beyond the usual checks) that would start a run outside the ship flow. Say which workflow would run and why, then wait. A go given once for a kind of run holds for the project; record it with the project's instructions. When the host builds the release artifacts on the tag, that run calls the gated build script of step 9; do not build them by hand as well. Runs cost minutes and can publish or deploy. One run proves one tree: where the host already runs the gate or the suites for a push or a pull request, the agent reads that result instead of running the same set again locally, and runs locally only what the host does not. The same holds between the host's own runs: where the merge build and the tag build run the gate on the same commit, or a host run only repeats the local runner, the agent names the repeat to the owner with a proposal (the tag build reuses the merge build's result, or the host keeps the caches between runs), and the owner decides whether the workflow changes.

## Emergency fallback

If the platform build is down, a manual upload path may exist. It is used only with the owner's go, takes the same gate and the same version bump, and the next normal deploy must include the same commit, so the branch and the host never drift apart.
