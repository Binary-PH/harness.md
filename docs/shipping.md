# Shipping

The main branch is production: the host builds and installs every merge into it. Only finished, checked work goes there, and every deploy carries a version bump.

## The repository and its remote

Every step below assumes a git repository with a remote the owner controls. A project that is not yet in git goes into git before the first feature.

- **No repository yet.** `git init`, a `.gitignore` for the stack, a `.gitattributes` with `* text=auto eol=lf`, one first commit. Then the owner creates the remote, or gives you the access to create it; push main and set it as the upstream. That setup commit is the only one that goes straight to main and carries no version bump; everything after it follows the ten steps. An existing repository without a remote gets only the remote and the push; its uncommitted work goes through a branch.
- **Choosing a host.** A host qualifies when its free plan gives private repositories, pull requests with squash merge through an API, a credential that can be limited to one repository (a scoped token or a deploy key), and a way to take the repository, issues and pull requests elsewhere. Hosted services with a free plan meet this for most projects. A self-hosted server such as Gitea runs as one container and fits an owner who already runs a server with a public hostname and TLS; a server on a home machine is not reachable from a cloud sandbox. Name two or three options with the reason and let the owner pick.
- **Windows.** Git is not preinstalled. Install it with the system package manager or the official installer, open a new terminal and confirm with `git --version`. Set `core.autocrlf` to `false` and let `.gitattributes` decide line endings, so diffs and the text rules are the same on every machine.
- **A session on a cloud sandbox.** The sandbox is thrown away when the session ends. Clone at the start, commit small, push the branch at every checkpoint and before the session ends. Reach the remote through the tool's own repository integration where one exists; otherwise through a credential limited to that one repository (a scoped token or a deploy key) with the shortest life the host allows, stored in the tool's secret settings, never pasted in the chat.

## The ten steps

1. **Branch.** Work on `<agent>/<topic>`, never on main. Start from an up-to-date main (`git switch main && git pull --ff-only`).
2. **Bump the version.** A small change (a fix, wording, a dependency update) bumps the third number. A mid-size change (a feature, a new rule or page) bumps the second. A large change (a rewrite, a break in what users rely on) bumps the first. The numbers after the bumped one reset to 0: from 1.0.0 that is 1.0.1, 1.1.0 or 2.0.0. Update the manifest, the lock file where it carries the version, and the CHANGELOG. If an app changed, bump its version as well, in the same task. Static assets the project serves get the new version, or a content hash, in their file names, with every reference updated, so no cache serves a stale copy; a query string on an unchanged name is not enough.
3. **Run the checks once.** The changed-files code check before each commit (`check --changed --gate`), then the filtered suites since the last release tag (`test --since <tag>`, `docs/testing.md`; no tag yet means everything). BLOCKED means fix the environment and rerun the suites. Never rerun a suite that passed on identical code.
4. **Snapshot context**, if the project keeps memory or lessons in the repo (`docs/lessons.md`).
5. **Open a pull request** that says what changed, why, which test tier ran and what was skipped. No co-author line and no "generated with" line.
6. **Squash-merge** through the platform's API with the title `<title> (#<number>)`, only when the checks pass and the owner approved. Read the result: delete the branch only when the response says the merge happened. If the merge is refused, say so and stop.
7. **Verify.** Poll the production health endpoint until it shows the new version (usually one to two minutes). If it does not, read the deploy log before anything else. A failed build leaves the old version running; say which version is live.
8. **Tag** the merge commit `v<version>` (annotated) and push the tag, once the new version is live. A version that never went live is not tagged.
9. **Release artifacts** (installers, app packages) go through the gated build script only, never a hand-run build.
10. **Clean up.** No test server or database you started left running, one-off files deleted, lessons saved.

## Pull request text

```text
What changed: <one paragraph>
Why: <the request or the problem>
Tests: tier <1|2|3>; ran <suites>; skipped <suites and why>; code check: clean | <n> reviewed false positives
Version: <old> to <new>
```

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

The host runs its own workflows for a push of a working branch, for a pull request, for a merge into main and for the release tag of step 8; those runs are the ship flow and need no extra go. Everything else waits for the owner's word: a run started by hand, a rerun of a failed run on the host, a workflow file added or changed, and a push (any other tag, or a branch with a trigger of its own beyond the usual checks) that would start a run outside the ship flow. Say which workflow would run and why, then wait. A go given once for a kind of run holds for the project; record it with the project's instructions. When the host builds the release artifacts on the tag, that run calls the gated build script of step 9; do not build them by hand as well. Runs cost minutes and can publish or deploy.

## Emergency fallback

If the platform build is down, a manual upload path may exist. It is used only with the owner's go, takes the same gate and the same version bump, and the next normal deploy must include the same commit, so the branch and the host never drift apart.
