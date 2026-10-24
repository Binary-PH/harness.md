# Maintaining this repository

For anyone, person or agent, who changes this repository. Read it whole before any change. `HARNESS.md` applies here as it applies to any project; this page adds what is specific to the harness itself and settles what the owner has already decided.

## What this repository is

Plain Markdown only: no scripts, no workflow files, no remote includes. Public, MIT, author BinaryPH. The entry file is `HARNESS.md`; the pages under `docs/` hold the detail; the README is the public face; the CHANGELOG records every change.

## Rules of the text

1. **No other name.** No mention of and no link to any other project, paper, person, company or AI product, and no citation. The only links point at this repository. Tool names in the tools table and check-tool names in the gate are fine as plain text, and so is the one self-hosted git server named in the shipping page. Every sentence is written fresh; nothing is copied from anywhere, not even a phrase.
2. **No em dash, no en dash.** The normal dash is fine wherever a dash is needed.
3. **Register.** Third person for description, imperative for orders, short sentences, no evaluative phrases. The README does not say "you" or "your". Lists, not tables, in the README: a table with code-formatted names does not wrap on a phone.
4. **The README's install lines never change**: they are quoted in posts. The README names no version and gives no advice to pin a tag; everyone follows the latest release.
5. **The logo** stays at the top of the README. Every other image in the README is a PNG in `assets/`, with a light and a dark variant where it carries text, and alt text that says everything the image shows. No SVG, because its namespace line is a link the scan reports, and no badge served from another host. A rendered image keeps its source beside it in the repository, with the steps that render it (the loop diagram: `assets/harness-flow.md`); a change edits the source and renders the images again in the same commit, never the images alone.
6. **No secret** anywhere, in any file or any history.
7. **One order per situation.** Every file agrees with every other. A change that touches an order is followed, in the same change, by a read of the files that order interacts with (the page the entry file points to for that section, the README lines that summarise it, and any page that uses the same term), and any clash is resolved before the commit.

## Adding to the harness

A standing rule from the owner (2026-10-24): every request to add something (a technique, a rule, a page, a tool) is worked through these steps: the first five are settled before any text is written, the sixth before the commit.

1. **Name what it is for.** State the mistake it prevents or the gap it fills for an agent working on a real project. What prevents no recurring mistake and fills no gap is not added.
2. **Fit it to the logic.** Place it in what already exists: the moment it runs (what runs when, `HARNESS.md` section 1), the section and the page it belongs to, and the orders it touches. Extending an existing order comes before adding a new one, and a page read when its moment comes before the entry file.
3. **Take what helps, skip the rest.** Split a technique into its parts and keep only those that change what an agent does. The CHANGELOG names the parts left out and why.
4. **Settle open questions by research.** Where a choice is unclear, find how large engineering organisations and reputable open-source projects handle it, decide, and record the decision in the list below and in the CHANGELOG, naming no source (rule 1 above). A choice the list below already settles is not open; a question goes to the owner only when no one else can answer it.
5. **Keep it lean.** No mechanism the project's tools already provide is rebuilt; no rule adds a test, a scan or a review that runs again on identical input; the entry file grows only by what an agent would otherwise get wrong.
6. **Leave no conflict.** The reads of the release procedure's step 4 run before the commit, every finding is fixed or left with its reason, and rule 7 above holds for every order the addition touches.

## Versioning

The version is in the title of `HARNESS.md` and in `CHANGELOG.md`, and this repository follows the rule it gives projects: wording or a fix bumps the third number, a new rule or page or an order that changes bumps the second, a rewrite of the orders bumps the first, and the numbers after the bumped one reset to 0.

## The release procedure, for every change

1. **Branch** in a cloud session. On the owner's machine the owner may commit on main.
2. **Scans**, all clean before the commit:

   ```text
   git ls-files | grep -v '\.png$' | xargs env LC_ALL=C grep -nP '\xE2\x80[\x93\x94]'   # em and en dash: no hit
   git ls-files | grep -v '\.png$' | xargs grep -noP 'https?://[^ )`>]+'              # links: only this repository
   ```

   The C locale makes the dash pattern match bytes; under a UTF-8 locale the same pattern finds nothing and reports a false clean. Links point only at this repository's own address, the one in the README. Then a grep for every product, brand, font or project name the change could have introduced, and a read of the diff for anything that looks like a key or a token.
3. **Lint**: `npx --yes markdownlint-cli2 "**/*.md"` with `.markdownlint.jsonc` from the repository. If it cannot run where you are, say so in the pull request; do not skip it silently.
4. **Cold read**: a fresh reader (a subagent, or you after a break) reads `HARNESS.md` alone and names the version, the seven gate steps, the three tiers and the FAILED, FLAKY, BLOCKED and not-run split with their exit codes. A new page gets a cold read of its own, with questions a user of that page would ask. A change to the gate or the runner also gets an implementation read: a fresh agent plans, without writing code, how it would build them for two different projects, and reports what would stop or mislead it. The rounds are set before the reads start (the reads, one fix round, one read of that round's diff), and points left after them that are only wording are checked by search. A change that leaves the text of `HARNESS.md` as it was, apart from the version in its title, needs no new cold read of it, since its text is the same input.
5. **Bump the version** in the `HARNESS.md` title and write the CHANGELOG entry, in the same commit.
6. **Commit** as `<version>: <summary>`, author BinaryPH. In a session that is not the owner's machine, set the author before the first commit: the name BinaryPH and the email the existing commits carry (`git log -1 --format=%ae`). No co-author trailer, no generated-with line, no session link, in the commit or in the pull request. A commit body, when there is one, says what changed and why.
7. **Pull request**: what changed, why, which checks ran and which were skipped, the version change. No trailer of any kind.
8. **The owner approves and merges** (ship step 6). After the squash-merge: an annotated tag `v<version>` on the merge commit, pushed, and the branch deleted, by whoever still has the repository open, the cloud session if it is, else the owner. A cloud session tends to skip both; the owner checks for them. A release is not finished until its tag is pushed: an agent's update check reads the newest tag. Before any change, `git ls-remote --tags origin` shows the tag for the version in the `HARNESS.md` title on main; a missing one is created on that merge commit first, which is not moving a tag.
9. **Never** a workflow file, a force push, a moved tag or a deleted branch that is not yours, without the owner's word.

## Decisions already taken

Do not reopen these; change them only when the owner says so.

- Simple over complete: the entry file stays short and holds only the orders an agent would otherwise get wrong, each pointing to the page to read when its situation comes; a mechanism the project's tools already provide (result caching, affected-test selection, incremental checks) is used, not rebuilt by hand. The owner asked for this on 2026-10-24 ("simple yet effective, not overengineered"); 2.0.0 follows the practice of reputable agent harnesses, because a long list of orders is followed less well than a short one.
- Platform analyzer warnings block only where the tools table says so; linter warnings are reported. Warnings hold style and the few noisy rules the linter policy names: the rules the linter itself classes as catching bugs are errors, and formatting is the formatter's check, which blocks.
- No change loosens the lint, coding or security standards: each check stays or tightens, and new wording only clarifies (owner, 2026-10-25). A choice that would loosen one waits for the owner.
- What the gate blocks is a minimum: a project's stricter setting stays and is never lowered to match the harness.
- A dependency finding without a severity score takes the highest score among its aliases; with none, it blocks until reviewed; a malicious-package advisory always blocks and goes on no list. A changed lock file is scanned at the commit; every lock file again before the merge, for new advisories.
- One rerun rule: never a suite that passed on identical code; once, at once, for a step on the known-flaky list, which reads FLAKY on a pass and FAILED on a second failure; after a fix for a failed or blocked step. The pass record holds in every tier, the expensive tier and a release run included: a hash per suite over its files, its own filters entry, the runner, the setup script and the manifests and lock files of its folders, with the version left out, decides whether it runs, and the test tool's own result cache or affected-test mode only narrows the tests inside; only a suite marked `uncached` and `--no-cache` at the owner's ask bypass it, and an `uncached` suite runs only in the expensive tier and when the owner asks. No test runs twice for the same result, and no gate step either, apart from the release build's full gate: the gate checks each commit's files; before the merge it runs only what can have a new answer (the dependency scan, and the whole-project type check and type-aware lint); the release build's full gate is the one run over every file, by design.
- Text is readable on its background in every user interface, with one theme or several: at least 4.5 to 1, and 3 to 1 for large text, icons, field borders and focus outlines; text over an image, a video or a gradient gets a backing. The desktop browser profile measures it in its existing run.
- The planning stage researches how large engineering organisations and reputable open-source projects solve the problem before the approach is chosen. The harness names none of them; a project's plan names its sources with their links.
- A push of a working branch, a pull request, a merge into main and the release tag are the ship flow and need no extra go; everything else on the host waits for the owner.
- No order repeats on every prompt: setup once per project, the reading once per session, a question runs nothing, a change runs what it reaches, and the ship flow runs once per task. One task, one branch, one pull request; prompts that continue a task add to its branch, and tasks stay small and merge soon, as short-lived branches and small batches are the practice of large engineering organisations.
- A lesson proved wrong is retired by renaming its file with the date, never removed and never edited.
- A step BLOCKED by a cause outside the agent's reach does not pass; the owner decides between waiting and merging with that step named in the pull request text as not proved.
- Exit codes of the test runner: `0` passed (skipped and FLAKY listed), `1` FAILED, `2` preflight, `3` blocked or not run, so a caller tells the code from the environment by the code alone.
- "Nothing else is downloaded" excepts the runtime, the database engine, git, an authorized security check's tools and a newer release of the harness itself.
- A project's copy of the harness is brought to the latest release by the agent in the first report of each session (the release tags read with git, then a depth-1 clone). No go is needed and no switch turns it off: everyone follows the latest release.
- The expensive tier runs on events, never on a calendar: before a merge that bumps the first or second number of the version or touches payments or billing, and when the owner asks. The full security review of the code runs with it; the checks of the deployed surface run once that release is live, and after a change to DNS, hosting, certificates or mail settings. A project's copy brought to a new release ships as its own pull request and bumps the project's third number, as a dependency update does.
- The version bump is by the size of the change, as above.
- Served assets carry their version in the file name; a query string is not enough.
- The README is lists, in the plain register, with the install lines fixed and no version named.
- A project's instruction file points at the harness without a version number; the version followed is the one in the title of the file the agent reads.
- A saved key or other secret shown in a user interface, and every password field, is hidden by default with a toggle beside it to show it; every open of the view starts hidden.
- A web system, web app or site is tested on a Windows desktop in every browser engine, Android, iPhone and the in-app browsers of the apps users open links from, on both phones: the desktop in the default engine and Android on every change, the other profiles once before a merge whose change reaches the front end and in the expensive tier.
- One self-hosted git server is named as the example; hosted services stay unnamed.
- The playbooks for 3D renders, interactive web pages, motion graphics and video, and search visibility each apply to their kind of project only, and the search page applies only to what the public reaches through search.
- The security review of the code reads source and runs code in a sandbox only; it never touches the deployed system, which stays with the three levels of the security posture page. Only a confirmed finding carries a severity; a needs validation record names its missing fact and carries none; a rejected record is kept. A finding is checked by a reader who did not hunt it: another agent, or where the tool has none, the finder in a later pass from the record alone, as decided with the page in 1.6.0.

## Memory and the inbox

This repository keeps no memory snapshot and no inbox file. A decision about the harness is recorded in the list above and in the CHANGELOG, in the same change that applies it. A lesson about maintaining the harness goes into the owner's memory on the owner's machine; a cloud session writes it into the pull request text instead.
