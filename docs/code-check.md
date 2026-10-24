# Code check gate

One command reports every problem a tool can find in the code and stops the build when a blocking problem exists. The build, package and release scripts call it themselves, so nobody can forget it. This page says what the command runs, in what order, when, and what blocks.

## The command

Build it once per project, in the language of the project's main server or build tooling (one language for the checker, even when the project has several), and name it plainly (for example `tools/check`). It takes:

- `[targets]`: which parts of the project to check (server, web, app). No target means all.
- `--gate`: exit non-zero when any blocking row exists. Without it the same report prints and the exit code is 0, so the command is safe to run at any time.
- `--uncommitted`: check only the files the next commit changes, untracked ones included.
- `--merge`: run what can have a new answer before a merge: the dependency step over every lock file, and the type check and the type-aware lint rules over the whole project.
- `--fast`: skip the slow static security analysis for a quick local loop. Never in the gate: the command refuses `--fast` together with `--gate`.

The report is one row per tool: the tool, its status, the count of findings, and the path to its JSON output. It is written under the folder the project names for installs and runs, not in the source tree.

## Steps, in order

| # | Step | Finds | How | Blocks when |
| - | ---- | ----- | --- | ----------- |
| 1 | Type check | type errors | the language's type checker over the whole project, strict settings, no output files | any error, or the checker did not run |
| 2 | Linter | bugs and unsafe patterns, not style | the language's linter with the policy below, JSON output | an error-level finding; warnings are reported only |
| 3 | Static security analysis | injection, cross-site scripting, framework pitfalls | a multi-language static analyser (Semgrep or equivalent) with the registry packs for the language and, where the project has what they cover, the web top-ten risks, cross-site scripting, SQL injection and the security audit set (secrets are step 4's); metrics off; a timeout per rule; dependency, build, data and vendored folders and minified files excluded | an ERROR-severity finding, or no readable output |
| 4 | Secrets | keys, tokens, passwords, private keys | a secrets scanner (Gitleaks or equivalent) over the tracked files, and over the git history once, when the gate is first built (after that, every commit's files are scanned before they enter it), with redacted output and the exit code ignored so the gate decides itself; the allow-list holds only dependency folders, build output (the build scripts scan it themselves, below), data folders, the ignored env file and files that are public by design | any finding |
| 5 | Dependency vulnerabilities | known vulnerable versions | a vulnerability scanner (OSV-Scanner or equivalent) over every lock file | a finding with a severity score of 7.0 or higher; lower scores are reported |
| 6 | Platform analyzer | errors the platform's own analyzer knows | the analyzer from the tools table (`docs/tools-by-language.md`): the Dart analyzer for Flutter, the WordPress standards' security sniffs for WordPress, and so on | errors, or no parsed output; warnings only where the tools table says so |
| 7 | Text rules | em dash (U+2014) and en dash (U+2013); the normal dash (hyphen-minus, U+002D) is allowed and never scanned | read every tracked text file; skip binaries (any NUL byte) and vendored folders; write the code points in the checker's source so it passes its own check | any hit; the report tells where and how to rewrite the sentence, with the normal dash where a dash is needed |

Run every step even when an earlier one failed, so one report shows everything. A step for a language the project does not have is left out, not reported as missing.

## Two rules that never let a check pass silently

1. **A tool that is not installed** is a row written `NOT INSTALLED` with the setup hint next to it. It is never skipped and never counted as a pass.
2. **A tool that ran but produced no output the checker can read blocks** (exit `1`, or `3` when its log shows an environment cause). Silence is not a pass.

A tool that could not run (not installed, or stopped by an environment cause such as a network failure) is reported apart from findings and exits `3`, so a caller tells the code from the environment by the code alone.

## When it runs

Each step once at each moment:

- **Before each commit**: `check --uncommitted --gate`. Only the steps the changed files reach run, by kind of file:
  - code files: the type check of that language over all its files (a change can break a file it did not touch), the linter, the static security analysis and the text rules;
  - platform files: the platform analyzer;
  - every changed file: the secrets scan;
  - a lock file or a manifest, other than in its version field: the type check and the linter of that language, since a new dependency can break the code that uses it; the dependency scan waits for the merge;
  - shell scripts, container files and Markdown: their linters (`docs/tools-by-language.md`) and the text rules;
  - other text files, docs included: the text rules.

  A change to the check command, a tool's configuration or the setup script runs the full gate instead, since it can change any answer.
- **Before the merge**: `check --merge --gate`: the dependency step over every lock file, since a new advisory appears without a change to the code, and the type check and the type-aware lint rules over the whole project, since main may have moved and a changed type can break a file nobody touched. The other steps already ran on each commit's files and do not run again.
- **In the build and package scripts**: the full gate, `check --gate`, once per release build. It is the one run over every file, by design: the rules and the advisories it applies may be newer than the commits it covers.

The command keeps no cache of its own; the caches the tools already have (an incremental type check, a linter's cache) speed it up.

## Findings accepted on adoption

A project that takes on the harness with many findings may, with the owner's go, record them in a list beside the check command (`known-findings.json`: the tool, the rule, the file and a fingerprint of the flagged line). A finding on the list is reported, not blocking; it leaves the list when it is fixed, and the list only shrinks. Every finding not on it blocks as usual. Where a tool has its own way to record such a list, the command uses it.

## Reviewed false positives

A finding that was reviewed and is not a problem gets two things on the spot: a comment on the line above, `// Reviewed (code check): <why it is safe>`, and the tool's own suppression marker on the line. Never a blanket disable for a file or a rule, and never a suppression without the comment: the review of the diff treats one as a check made easier to pass (`docs/code-review.md`).

## Linter policy

- **Errors** (block): rules that catch bugs and unsafe code. Self comparison, a loop condition that never changes, an unreachable loop, a callback that must return a value, a return in a constructor, a return in a promise executor, `eval` and implied `eval`, an assignment inside a condition (except in parentheses).
- **Warnings** (reported, never block): style and tidiness. Unused variables, empty blocks, loose equality, variables that could be constants.
- **Type-aware rules as errors** in typed languages: no explicit `any`, no floating promises, no misused promises (a void-returning callback is allowed), `await` only on things that can be awaited. Relax these to warnings in UI code and in tests, where the noise outweighs the catch.
- **Security plugin** of the linter on, with its noisiest rule (object injection) off and the file-name rule as a warning.
- **UI framework rules**: the rules-of-hooks kind as errors, exhaustive-dependencies as a warning.
- **Generated output is never linted.** Local tool scripts get the bug rules but no security gate.
- **One tool, one run.** A tool that fills more than one row runs once, and its findings are sorted into the rows by source. In a language whose analyzer bundles the type check, the lint and the platform analysis (Dart), one run fills rows 1, 2 and 6: errors to row 1, lint-rule findings to row 2 under the linter policy, and the analyzer's own warnings to row 6, where they block. The same holds for `phpcs` with the WordPress standards (the security sniffs to row 6, the rest to row 2) and for a bug finder that serves the linter and the platform rows. A tool whose work another row already does is not run at all: the tools table says none for that row.

## How the build scripts enforce the gate

- The package script runs `check <targets> --gate`, then the runner's release run for the same targets (`test <targets> --release`, `docs/testing.md`), then the packaging steps: the build, a scan of the built output for secrets and for source maps (a key inlined at build time and a shipped `.map` file or `sourceMappingURL` comment both block), and the archive. If the checker file is missing, the script stops rather than packaging. A `--skip-check` option may exist for the one case where the checker itself cannot run, is used only with the owner's go, and prints a loud notice when used.
- The app build script runs `check <app> --gate`, then the runner's release run for the app, then the release build and the same scan of the built output, then copies the artifact to `dist/<app>-<version>`. It stops at the first step that does not pass.
- A release that bypassed the gate is reported as such in its notes.

## Exit codes

- `0`: clean, or run without `--gate`.
- `1`: a blocking finding exists (with `--gate`).
- `2`: the checker itself could not run (missing configuration, no git repository).
- `3`: no blocking finding, but a tool could not run (with `--gate`).

## What to install

The tools table (`docs/tools-by-language.md`) names, per language, the type checker, linter, formatter, platform analyzer and lock file. Install only the tools for the languages the project uses, and pin their versions in the setup script so a drift shows up in the preflight (`docs/testing.md`).
