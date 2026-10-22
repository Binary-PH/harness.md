# Maintaining this repository

For anyone, person or agent, who changes this repository. Read it whole before any change. `HARNESS.md` applies here as it applies to any project; this page adds what is specific to the harness itself and settles what the owner has already decided.

## What this repository is

Plain Markdown only: no scripts, no workflow files, no remote includes. Public, MIT, author BinaryPH. The entry file is `HARNESS.md`; the pages under `docs/` hold the detail; the README is the public face; the CHANGELOG records every change.

## Rules of the text

1. **No other name.** No mention of and no link to any other project, paper, person, company or AI product, and no citation. The only links point at this repository. Tool names in the tools table and check-tool names in the gate are fine as plain text, and so is the one self-hosted git server named in the shipping page. Every sentence is written fresh; nothing is copied from anywhere, not even a phrase.
2. **No em dash, no en dash.** The normal dash is fine wherever a dash is needed.
3. **Register.** Third person for description, imperative for orders, short sentences, no evaluative phrases. The README does not say "you" or "your". Lists, not tables, in the README: a table with code-formatted names does not wrap on a phone.
4. **The README's install lines never change**: they are quoted in posts. The README names no version and gives no advice to pin a tag; everyone follows the latest release.
5. **The logo** stays at the top of the README.
6. **No secret** anywhere, in any file or any history.
7. **One order per situation.** Every file agrees with every other. A change that touches an order is followed, in the same change, by a read of the files that order interacts with (the page the entry file points to for that section, the README lines that summarise it, and any page that uses the same term), and any clash is resolved before the commit.

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
4. **Cold read**: a fresh reader (a subagent, or you after a break) reads `HARNESS.md` alone and names the version, the seven gate steps, the three tiers and the FAILED, BLOCKED and not-run split. A new page gets a cold read of its own, with questions a user of that page would ask.
5. **Bump the version** in the `HARNESS.md` title and write the CHANGELOG entry, in the same commit.
6. **Commit** as `<version>: <summary>`, author BinaryPH. In a session that is not the owner's machine, set the author before the first commit: the name BinaryPH and the email the existing commits carry (`git log -1 --format=%ae`). No co-author trailer, no generated-with line, no session link, in the commit or in the pull request. A commit body, when there is one, says what changed and why.
7. **Pull request**: what changed, why, which checks ran and which were skipped, the version change. No trailer of any kind.
8. **The owner approves and merges** (ship step 6). After the squash-merge: an annotated tag `v<version>` on the merge commit, pushed, and the branch deleted, by whoever still has the repository open, the cloud session if it is, else the owner. A cloud session tends to skip both; the owner checks for them.
9. **Never** a workflow file, a force push, a moved tag or a deleted branch that is not yours, without the owner's word.

## Decisions already taken

Do not reopen these; change them only when the owner says so.

- Platform analyzer warnings block only where the tools table says so; linter warnings are reported.
- One rerun rule: never a suite that passed on identical code; once for a known flaky step; after a fix for a failed or blocked step; the expensive tier and a release run ignore the cache.
- A push of a working branch, a pull request, a merge into main and the release tag are the ship flow and need no extra go; everything else on the host waits for the owner.
- A lesson proved wrong is retired by renaming its file with the date, never removed and never edited.
- "Nothing else is downloaded" excepts the runtime, the database engine, git and an authorized security check's tools.
- The version bump is by the size of the change, as above.
- Served assets carry their version in the file name; a query string is not enough.
- The README is lists, in the plain register, with the install lines fixed and no version named.
- A saved key or other secret shown in a user interface, and every password field, is hidden by default with a toggle beside it to show it; every open of the view starts hidden.
- A web system, web app or site is tested on a Windows desktop in every browser engine, Android, iPhone and the in-app browsers of the apps users open links from, on both phones: the desktop in the default engine and Android on every change, the other profiles in the expensive tier.
- One self-hosted git server is named as the example; hosted services stay unnamed.
- The playbooks for 3D renders, interactive web pages, motion graphics and video, and search visibility each apply to their kind of project only, and the search page applies only to what the public reaches through search.
- The security review of the code reads source and runs code in a sandbox only; it never touches the deployed system, which stays with the three levels of the security posture page. Only a confirmed finding carries a severity; a needs validation record names its missing fact and carries none; a rejected record is kept. A finding is checked by a reader who did not hunt it.

## Memory and the inbox

This repository keeps no memory snapshot and no inbox file. A decision about the harness is recorded in the list above and in the CHANGELOG, in the same change that applies it. A lesson about maintaining the harness goes into the owner's memory on the owner's machine; a cloud session writes it into the pull request text instead.
