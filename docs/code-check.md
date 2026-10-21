# Code check gate

One command reports every problem a tool can find in the code and stops the build when a blocking problem exists. The build, package and release scripts call it themselves, so nobody can forget it. This page says what the command runs, in what order, and what blocks.

## The command

Build it once per project, in the language of the project's main server or build tooling (one language for the checker, even when the project has several), and name it plainly (for example `tools/check`). It takes:

- `[targets]`: which parts of the project to check (server, web, app). No target means all.
- `--gate`: exit non-zero when any blocking row exists. Without it the same report prints and the exit code is 0, so the command is safe to run at any time.
- `--changed`: check only the files changed against the merge base with main, plus uncommitted and untracked files (see below).
- `--fast`: skip the slow static security analysis for a quick local loop. Never in the gate: the command refuses `--fast` together with `--gate`.

The report is one row per tool: the tool, its status, the count of findings, and the path to its JSON output. The full run writes to `reports/latest/` (a `summary.txt` plus one JSON file per tool) under the folder the project names for installs and runs, not in the source tree.

## Steps, in order

| # | Step | Finds | How | Blocks when |
| - | ---- | ----- | --- | ----------- |
| 1 | Type check | type errors | the language's type checker over the whole project, strict settings, no output files | any error, or the checker did not run |
| 2 | Linter | bugs and unsafe patterns, not style | the language's linter with the policy below, JSON output | an error-level finding; warnings are reported only |
| 3 | Static security analysis | injection, cross-site scripting, hard-coded secrets, framework pitfalls | a multi-language static analyser (Semgrep or equivalent) with the registry packs for the language, the web top-ten risks, secrets, cross-site scripting, SQL injection and the security audit set; metrics off; a timeout per rule; dependency, build, data and vendored folders and minified files excluded | an ERROR-severity finding, or no readable output |
| 4 | Secrets in what ships | keys, tokens, passwords, private keys | a secrets scanner (Gitleaks or equivalent) over the folders that ship (server code, app source, assets), redacted output, exit code ignored so the gate decides itself; the allow-list holds only dependency folders, build output, data folders, the ignored env file and files that are public by design | any finding |
| 5 | Dependency vulnerabilities | known vulnerable versions | a vulnerability scanner (OSV-Scanner or equivalent) over every lock file | a finding with a severity score of 7.0 or higher; lower scores are reported |
| 6 | Platform analyzer | errors the platform's own analyzer knows | the analyzer from the tools table (`docs/tools-by-language.md`): the Dart analyzer for Flutter, `phpcs` with the WordPress standards for WordPress, and so on | errors, or no parsed output; warnings only where the tools table says so |
| 7 | Text rules | em dash (U+2014) and en dash (U+2013); the normal dash (hyphen-minus, U+002D) is allowed and never scanned | read every tracked text file; skip binaries (any NUL byte) and vendored folders; write the code points in the checker's source so it passes its own check | any hit; the report tells where and how to rewrite the sentence, with the normal dash where a dash is needed |

Run every step even when an earlier one failed, so one report shows everything.

## Two rules that never let a check pass silently

1. **A tool that is not installed is a blocking row**, written `NOT INSTALLED` with the setup hint next to it. It is never skipped and never counted as a pass.
2. **A tool that ran but produced no output the checker can read also blocks.** Silence is not a pass.

## Changed-files mode

`--changed` takes the files changed against the merge base with main (`git diff --name-only <merge-base>`; before the first commit on a branch that is the working-tree diff) plus uncommitted and untracked files that are not ignored, and sorts them into buckets by kind: code files, platform files, files that may hold secrets, lock files, text files. Only the steps a bucket reaches run:

- code files: type check (when a file of that kind changed), linter, static security analysis, text rules;
- platform files: the platform analyzer (when more than about 60 files changed, run the whole project);
- any file in the folders that ship: one secrets scan per changed file;
- lock files: the dependency scan, only for the lock files that changed;
- text files, including docs: the text rules.

Each changed-mode run writes to its own folder (`reports/changed-<pid>/`), so two runs never collide, and the folder is kept only when something was found.

## Reviewed false positives

A finding that was reviewed and is not a problem gets two things on the spot: a comment on the line above, `// Reviewed (code check): <why it is safe>`, and the tool's own suppression marker on the line. Never a blanket disable for a file or a rule, and never a suppression without the comment.

## Linter policy

- **Errors** (block): rules that catch bugs and unsafe code. Self comparison, a loop condition that never changes, an unreachable loop, a callback that must return a value, a return in a constructor, a return in a promise executor, `eval` and implied `eval`, an assignment inside a condition (except in parentheses).
- **Warnings** (reported, never block): style and tidiness. Unused variables, empty blocks, loose equality, variables that could be constants.
- **Type-aware rules as errors** in typed languages: no explicit `any`, no floating promises, no misused promises (a void-returning callback is allowed), `await` only on things that can be awaited. Relax these to warnings in UI code and in tests, where the noise outweighs the catch.
- **Security plugin** of the linter on, with its noisiest rule (object injection) off and the file-name rule as a warning.
- **UI framework rules**: the rules-of-hooks kind as errors, exhaustive-dependencies as a warning.
- **Generated output is never linted.** Local tool scripts get the bug rules but no security gate.
- **One analyzer, three rows.** In a language whose analyzer bundles the type check, the lint and the platform analysis (Dart), one run fills rows 1, 2 and 6; map the rule classes above to that analyzer's lint set with its strict modes on. Its findings are sorted by source: errors to row 1, lint-rule findings to row 2 under the linter policy, the analyzer's own warnings to row 6, where they block.

## How the build scripts enforce the gate

- The package script runs `check <targets> --gate`, then the build (type check, bundle), then the archive. If the checker file is missing, the script stops rather than packaging. A `--skip-check` switch may exist for the one case where the checker itself cannot run, is used only with the owner's go, and prints a loud notice when used.
- The app build script runs `check <app> --gate`, then the unit tests, then the release build, then copies the artifact to `dist/<app>-<version>`. It stops at the first failing step.
- A release that bypassed the gate is reported as such in its notes.

## Exit codes

- `0`: clean, or run without `--gate`.
- `1`: a blocking row exists (with `--gate`).
- `2`: the checker itself could not run (missing configuration, no git repository).

## What to install

The tools table (`docs/tools-by-language.md`) names, per language, the type checker, linter, formatter, platform analyzer and lock file. Install only the tools for the languages the project uses, and pin their versions in the setup script so a drift shows up in the preflight (`docs/testing.md`).
