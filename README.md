<p align="center">
  <img src="assets/binaryph-logo.png" width="220" alt="BinaryPH">
</p>

# BinaryPH AI Harness

Standing orders for AI coding agents, in plain Markdown.

A harness is everything around an AI coding agent that decides how it works: the instructions it reads, the checks it must pass, the way it tests, the way it ships, and the lessons it keeps. BinaryPH AI Harness is one such harness, written as plain Markdown, that any agent can follow in any language or stack. The agent reads one file, builds the project's check command and test runner from the specs, and applies the same rules on every project.

## Contents

- [What a harness is made of](#what-a-harness-is-made-of)
- [Install](#install)
- [First use](#first-use)
- [Files](#files)
- [Trust](#trust)
- [Versioning](#versioning)
- [License](#license)

## What a harness is made of

- **Instructions** (`HARNESS.md`): the orders the agent reads first: working rules, decisions before coding, parity across copies of a product.
- **Code check gate** (`docs/code-check.md`, `docs/tools-by-language.md`): one command before every build, package or ship: type check, linter, static security analysis, secrets scan, dependency vulnerabilities, platform analyzer, text rules.
- **Test runner** (`docs/testing.md`): three tiers, path filters, no rerun on identical code, a preflight that prints the fix for every problem, a fresh database per run, FAILED (the code is wrong) apart from BLOCKED (the environment could not run it).
- **Ship flow** (`docs/shipping.md`): main is production: branch, version bump, checks once, pull request, squash-merge, health check, tag; served assets carry their version in the file name; no workflow run started by hand without the owner's go.
- **Security posture** (`docs/security.md`): passive, light and active checks of the deployed surface, and the rules before any check.
- **Memory** (`docs/lessons.md`): lessons with a cue, the lesson and the evidence; none without evidence; retired with a date, never deleted; bounded recall; links that show their evidence.
- **Playbooks** (`docs/playbooks.md` and the four pages under `docs/playbooks/`): decisions before coding, WordPress, 3D renders from an image, interactive web pages, motion graphics and video, search visibility for web apps and sites.

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

Some tools can import a file with a line that reads `@HARNESS.md` (from a subfolder, `@<subfolder>/HARNESS.md`); check the tool's documentation. A copy is frozen: replace it to take a newer release.

## First use

In the first session on a project, the agent:

1. Reads `HARNESS.md` and states which version it follows.
2. Puts the code in git with a remote the owner controls, if it is not there yet.
3. Sets up the environment with the best tools for the job, only those that fit the project's languages, pinned.
4. Writes the project's own check command and test runner from the specs, in the project's language, and wires the check into the build scripts.
5. Runs the gate once and fixes what it finds before any feature work.
6. Keeps lessons with evidence as it works.

## Files

- `HARNESS.md`: the entry file: imperative orders, about ten minutes to read.
- `docs/code-check.md`: the gate, step by step, with what blocks.
- `docs/testing.md`: tiers, path filters, isolation, lanes, hashing, preflight, database hygiene, verdicts, the A/B rule.
- `docs/shipping.md`: the repository and its remote, the ten ship steps, pull request text, dependency update bot pull requests, workflow runs on the host.
- `docs/security.md`: passive, light and active checks of the deployed surface, and the rules before any check.
- `docs/lessons.md`: the lesson format, memory, snapshot and inbox, recall, links and retirement.
- `docs/playbooks.md`: decisions before coding; WordPress; pointers to the four playbooks.
- `docs/playbooks/3d-renders.md`: a 3D model from a reference image, built from a spec in passes and judged against the image.
- `docs/playbooks/interactive-web.md`: scroll-driven and 3D pages, accessible and light.
- `docs/playbooks/motion-video.md`: a video from code: one timeline, frames from time alone, a review loop over contact sheets, generated sound.
- `docs/playbooks/web-seo.md`: search visibility for a web app or site: the first HTML, crawl and index, speed and images, page content, structured data, checks by kind of site, the audit.
- `docs/tools-by-language.md`: the tool table per language and the cross-language tools.
- `CHANGELOG.md`: every change to the harness.
- `CONTRIBUTING.md`: how this repository is maintained: the rules of the text, the release procedure, the decisions already taken.

## Trust

- These files are plain Markdown: no scripts, no remote includes.
- Releases are tagged. The CHANGELOG lists every change.
- The repository passes its own text rules before each release: no em dash, no en dash (the normal dash is fine), no link except to this repository, no secret.

## Versioning

The version is in the title of `HARNESS.md`, so an agent that reads the file can say which version it follows, and in `CHANGELOG.md`. This repository follows the rule it gives projects: a small change (wording, a fix) bumps the third number, a mid-size change (a new rule or page, or an order that changes) bumps the second, a large change (a rewrite of the orders, or a break in how the harness is used) bumps the first, and the numbers after the bumped one reset to 0. From 1.0.0 that is 1.0.1, 1.1.0 or 2.0.0.

## License

MIT. Author: BinaryPH.
