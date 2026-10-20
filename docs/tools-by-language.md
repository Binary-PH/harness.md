# Tools by language

Install the best tool for each need: the row for the languages the project uses, plus the cross-language rows, or an equivalent that fits the project better, with the reason noted. Nothing outside those rows is downloaded, apart from what the project itself needs to run (the runtime, the database engine, git) and what an authorized security check needs (`docs/security.md`). A cell that says none leaves that gate row out for the language; it is not a `NOT INSTALLED` row. Pin every version in the project's setup script so the preflight can report a drift.

## Per language

| Language | Type check | Linter | Formatter | Platform analyzer | Test runner | Lock file for the dependency scan |
| -------- | ---------- | ------ | --------- | ----------------- | ----------- | --------------------------------- |
| TypeScript, JavaScript | the TypeScript compiler with `--noEmit`, strict | ESLint (flat config) with the typed rules and the security plugin | Prettier | framework rule sets inside ESLint | the project's runner (the built-in test runner, or Vitest or Jest) | `package-lock.json`, `pnpm-lock.yaml` or `yarn.lock` |
| Python | mypy or pyright, strict | ruff (error, pyflakes, bugbear and security rule sets) | ruff format or black | bandit | pytest | `uv.lock`, `poetry.lock` or a pinned `requirements.txt` |
| Go | the compiler and `go vet` | golangci-lint with staticcheck | gofmt | gosec | `go test` | `go.sum` |
| Dart, Flutter | the Dart analyzer with strict modes in `analysis_options.yaml` | the analyzer's lint set for Flutter | `dart format` | `flutter analyze` (the analyzer's own warnings block) | `flutter test` | `pubspec.lock` |
| PHP, WordPress | phpstan (level 6 or higher) | phpcs with the PHP compatibility standard, plus the WordPress coding standards for WordPress projects | phpcbf | the WordPress standards' security sniffs (escaping, nonces, prepared statements) | phpunit | `composer.lock` |
| Rust | `cargo check` | clippy | rustfmt | `cargo audit` | `cargo test` | `Cargo.lock` |
| Java, Kotlin | the compiler with warnings as errors | Checkstyle and SpotBugs, or detekt and ktlint | Spotless, or ktlint | SpotBugs with the security plugin | JUnit | the build tool's lock file |
| Ruby | sorbet, or none | rubocop | rubocop autocorrect | brakeman for web apps | rspec or minitest | `Gemfile.lock` |
| C# | the compiler with nullable on and warnings as errors | the built-in analyzers | `dotnet format` | Security Code Scan | `dotnet test` | `packages.lock.json` |
| Swift | the compiler | SwiftLint | swift-format | none | XCTest | `Package.resolved` |

## Cross-language (every project, each row only when it has such files)

| Need | Tool class | Example |
| ---- | ---------- | ------- |
| Static security analysis | multi-language rule-based analyser with a public rule registry | Semgrep |
| Secrets in the tree and in history | secrets scanner with a redacted report | Gitleaks |
| Known vulnerable dependencies | lock-file vulnerability scanner with severity scores | OSV-Scanner |
| Shell scripts | shell linter | ShellCheck |
| Container files | container-file linter | hadolint |
| Markdown | markdown linter | markdownlint |
| Text rules | the em dash and en dash scan built into the gate | part of the check command |

The shell, container-file and Markdown linters run inside the linter step for the files they cover, under the linter policy.

## Install hints

- Put every install in one setup script per machine kind (developer machine, cloud session), with the version pinned next to each tool.
- Installs, caches, databases and test output go to the folder the project names for installs and runs, outside the source tree, never next to the source.
- Behind a TLS-intercepting proxy or antivirus, give the tools the proxy's certificate bundle through their environment variables instead of turning verification off.
- The preflight (`docs/testing.md`) compares installed versions with the pinned ones and warns on a drift.
