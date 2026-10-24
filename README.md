<p align="center">
  <img src="assets/binaryph-logo.png" width="220" alt="BinaryPH">
</p>

<h1 align="center">BinaryPH AI Harness</h1>

<p align="center">Standing orders for AI coding agents, in plain Markdown.</p>

<p align="center">
  <a href="#install">Install</a> ·
  <a href="#how-it-works">How it works</a> ·
  <a href="HARNESS.md">The orders</a> ·
  <a href="CHANGELOG.md">Changelog</a> ·
  <a href="CONTRIBUTING.md">Contributing</a> ·
  <a href="LICENSE">MIT license</a>
</p>

A **harness** is everything around an AI coding agent that decides how it works: the instructions it reads, the checks it must pass, the way it tests, the way it ships, and the lessons it keeps. **BinaryPH AI Harness** is one such harness, written as plain Markdown, that any agent can follow in any language or stack. The agent reads one file, builds the project's check command and test runner from the specs, and applies the same rules on every project.

It covers three stretches of every task:

- **Before the code**: the agent restates the ask and, for a feature or anything larger, researches how large engineering organisations and reputable open-source projects solve the same problem, reads the documentation of the versions in use, and writes a plan with success criteria a test can check, cut into thin slices.
- **Around the code**: one slice at a time with a failing test first, a gate of seven checks in one command, a test runner that runs only what a change reaches and never repeats a result it already has, a review of the diff that keeps every check as strict as it was, and a security review where a change crosses a trust boundary.
- **After the code**: a ship flow where main is production, a version and a tag on every release, a way back for a change that is hard to undo, and lessons kept only with evidence.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/harness-flow-dark.png">
    <img src="assets/harness-flow-light.png" width="100%" alt="The agent's loop for one task, in eight steps. 1 Read: HARNESS.md, whole, once a session; a copy updates itself to the latest release. 2 Plan: planning.md and playbooks.md; research how others solve it; success criteria and thin slices. 3 Build: HARNESS.md section 3; one slice at a time; a failing test first. 4 Check: code-check.md; seven steps in one command; nothing passes silently. 5 Test: testing.md; only what a change reaches; FAILED, BLOCKED and FLAKY kept apart. 6 Review: code-review.md; the agent reads its own diff; no check loosened, trust boundaries read. 7 Ship: shipping.md; main is production; version, tag and a way back. 8 Learn: lessons.md; lessons with evidence, recalled in the next session. A dashed arrow leads from Learn back to Plan for the next task.">
  </picture>
</p>

## Contents

- [How it works](#how-it-works)
- [What a harness is made of](#what-a-harness-is-made-of)
- [Install](#install)
- [First use](#first-use)
- [Files](#files)
- [Trust](#trust)
- [Versioning](#versioning)
- [Contributing](#contributing)
- [License](#license)

## How it works

Every task runs the same loop, and each step has its page:

1. **Read.** The agent reads `HARNESS.md` whole at the start of a session and states the version it follows. A copy kept in a project brings itself to the latest release first.
2. **Plan.** It restates the ask. For a feature, a new project or a change that is hard to undo, it also researches how others solve the same problem and writes a plan: goal, non-goals, success criteria, the approach, the options that lost, the risks with the way back. The plan is cut into thin vertical slices, the riskiest first. A new project, or a change that is hard to undo, waits for the owner's go; a small fix is its own one slice.
3. **Build.** One slice at a time. A change in behaviour starts with a test seen failing; a bug fix starts with the test that reproduces it.
4. **Check.** One command runs the type check, the linter, static security analysis, a secrets scan, the dependency scan, the platform analyzer and the text rules. A missing tool or an unreadable result blocks; a step already clean on identical files is skipped and says so.
5. **Test.** The runner runs only the suites a change reaches, never a suite that passed on identical code, and keeps FAILED (the code is wrong) apart from BLOCKED (the environment could not run it) and FLAKY (a listed flaky test that passed on its one rerun).
6. **Review.** The agent reads its own diff for correctness, plainness, fit, cost and leftovers, and for any check made easier to pass. A change across a trust boundary also gets a security review of the code. Both come before the checks that run before the merge, so a fix they bring is checked once.
7. **Ship.** Branch, version bump, checks once, pull request, squash-merge, health check, tag. A broken release is reverted through the same flow before it is debugged.
8. **Learn.** Up to three lessons per task, each with a cue and its evidence, recalled when a later task matches the cue.

## What a harness is made of

- **Instructions** (`HARNESS.md`): the orders the agent reads first: working rules, planning and building in slices, parity across copies of a product.
- **Planning** (`docs/planning.md`): the order of decisions before coding, research into how large organisations and reputable open-source projects do it, the documentation of the version in use, the plan, slices, decision records.
- **Code check gate** (`docs/code-check.md`, `docs/tools-by-language.md`): one command before every build, package or ship: type check, linter, static security analysis, secrets scan, dependency vulnerabilities, platform analyzer, text rules; no step run twice on identical files.
- **Test runner** (`docs/testing.md`): three tiers, path filters, no rerun on identical code, a preflight that prints the fix for every problem, a fresh database per run, FAILED, BLOCKED and FLAKY told apart by verdict and by exit codes that tell the code from the environment, how to write a test and debug a failure, and for a web app or site the browsers and devices every screen must pass.
- **Review of the diff** (`docs/code-review.md`): the tests first, five points, no check made easier to pass, a size a reader can hold, a second reader for a change that is hard to undo.
- **Ship flow** (`docs/shipping.md`): main is production: branch, version bump, checks once, pull request, squash-merge, health check, tag; changes a running version must survive and the way back; served assets carry their version in the file name; no workflow run started by hand without the owner's go.
- **Security posture** (`docs/security-posture.md`): passive, light and active checks of the deployed surface, and the rules before any check.
- **Security review of the code** (`docs/security-review.md`): a reader follows the code from where input enters to where it does damage; source and a sandbox only; a finding names the actor, the input, the control, the path, who is affected and the observed result; a second reader tries to disprove it; confirmed, needs validation and rejected kept apart; a coverage list as the claim.
- **Memory** (`docs/lessons.md`): lessons with a cue, the lesson and the evidence; none without evidence; retired with a date, never deleted; bounded recall; links that show their evidence.
- **Playbooks** (`docs/playbooks.md` and the four pages under `docs/playbooks/`): WordPress, 3D renders from an image, interactive web pages, motion graphics and video, search visibility for web apps and sites.

## Install

Two ways. The second is more reliable, because some agents treat a page fetched from the web as data rather than as orders.

**1. From the web.** Paste this to the agent; it fetches the file from this repository:

```text
Read HARNESS.md at https://github.com/Binary-PH/harness.md and follow it for this project.
```

**2. From a copy in the project.** Download this repository (clone it, or the zip from the releases page) and copy `HARNESS.md` and the whole `docs/` folder into the project; `HARNESS.md` points at the pages in `docs/`, so it does not work alone. Then add this line to the instruction file the agent reads at the start of every session (the tool's documentation names that file):

```text
Read HARNESS.md in this repo and follow it.
```

Some tools can import a file with a line that reads `@HARNESS.md` (from a subfolder, `@<subfolder>/HARNESS.md`); check the tool's documentation. A copy keeps itself current: in the first report of each session the agent checks for a newer release and, when there is one, replaces the copy and says what changed.

## First use

In the first session on a project, the agent:

1. Reads `HARNESS.md` and states which version it follows. The project's instruction file points at the harness without a version number, so the next session reads the latest release, or the copy in the project, which the agent brings to the latest release first.
2. Puts the code in git with a remote the owner controls, if it is not there yet.
3. Sets up the environment with the best tools for the job, only those that fit the project's languages, pinned.
4. Writes the project's own check command and test runner from the specs, in the project's language, and wires the check into the build scripts.
5. Runs the gate once and fixes what it finds before any feature work.
6. Plans each task before its code and builds it one slice at a time.
7. Keeps lessons with evidence as it works.

## Files

<details>
<summary>Every file in this repository, with what it holds</summary>

- `HARNESS.md`: the entry file: imperative orders, about ten minutes to read.
- `docs/planning.md`: how much planning a change needs, the order of decisions, the research, the plan's headings, slices, where the plan lives, decision records.
- `docs/code-check.md`: the gate, step by step, with what blocks and when a step is skipped on identical files.
- `docs/testing.md`: tiers, path filters, browsers and devices for a web app or site, isolation, lanes, hashing, writing tests, preflight, database hygiene, verdicts, debugging a failure, the A/B rule.
- `docs/code-review.md`: the review of the diff: when, tests first, five points, no check made easier to pass, size, a second reader.
- `docs/shipping.md`: the repository and its remote, the ten ship steps, pull request text, changes a running version must survive, the way back, dependency update bot pull requests, workflow runs on the host.
- `docs/security-posture.md`: passive, light and active checks of the deployed surface, and the rules before any check.
- `docs/security-review.md`: the security review of the code: when it runs, its rules, the six things a finding names, the three verdicts, severity, coverage, the classes of flaw, the report.
- `docs/lessons.md`: the lesson format, memory, snapshot and inbox, recall, links and retirement.
- `docs/playbooks.md`: what a kind of project adds: WordPress, and pointers to the four playbooks.
- `docs/playbooks/3d-renders.md`: a 3D model from a reference image, built from a spec in passes and judged against the image.
- `docs/playbooks/interactive-web.md`: scroll-driven and 3D pages, accessible and light.
- `docs/playbooks/motion-video.md`: a video from code: one timeline, frames from time alone, a review loop over contact sheets, generated sound.
- `docs/playbooks/web-seo.md`: search visibility for a web app or site: the first HTML, crawl and index, speed and images, page content, structured data, checks by kind of site, the audit.
- `docs/tools-by-language.md`: the tool table per language and the cross-language tools.
- `CHANGELOG.md`: every change to the harness.
- `CONTRIBUTING.md`: how this repository is maintained: the rules of the text, the release procedure, the decisions already taken.
- `assets/`: the logo and the diagram above, in a light and a dark version.

</details>

## Trust

- These files are plain Markdown: no scripts, no remote includes.
- Releases are tagged. The CHANGELOG lists every change.
- The repository passes its own text rules before each release: no em dash, no en dash (the normal dash is fine), no link except to this repository, no secret.

## Versioning

The version is in the title of `HARNESS.md`, so an agent that reads the file can say which version it follows, and in `CHANGELOG.md`. This repository follows the rule it gives projects: a small change (wording, a fix) bumps the third number, a mid-size change (a new rule or page, or an order that changes) bumps the second, a large change (a rewrite of the orders, or a break in how the harness is used) bumps the first, and the numbers after the bumped one reset to 0. From 1.0.0 that is 1.0.1, 1.1.0 or 2.0.0.

## Contributing

Changes to the harness follow `CONTRIBUTING.md`: the rules of the text, the release procedure with its scans and its lint, and the decisions already taken.

## License

MIT. Author: BinaryPH.
