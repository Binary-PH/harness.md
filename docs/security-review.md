# Security review of the code

The code check gate (`docs/code-check.md`) runs tools over the code, and the security posture page (`docs/security.md`) looks at the deployed surface. This page covers what neither finds: a reader following the code from the place an outsider's input enters to the place it does damage, looking for the check that is missing, wrong, or on the wrong path. Authorization that tests the wrong permission, a workflow whose steps can be skipped, an export that carries more than the exporter may see: no scanner reports these, and a reader finds them.

## When a review runs

- **Scoped, before the pull request**, when a change touches a trust boundary: sign-in, sessions or tokens, permissions, payments or quantities, file paths or shell commands, deserialization, a new place where outside input enters, a workflow or build file, or a model-driven agent and its tools. The review covers the diff and the paths it reaches. Its map covers only the boundaries the diff touches, its coverage list has only their rows, and its report is the line in the pull request text (which boundaries were read, what was found) plus the record of each finding.
- **Full**, over the whole project, on any of three occasions: the owner asks, a release is large or carries money, or the expensive test tier runs (`docs/testing.md`).
- A review never changes the code it reviews. A fix is a change of its own, through the gate and the tests like any other, with a regression test that fails before the fix and passes after it.

## Rules of the review

- **Source and a sandbox only.** The review reads the repository and runs the project's code only in a sandbox: no network, an empty environment holding only the variables the check needs, the source tree read only, writes only to a scratch folder, and caps on time, memory and processes. Made-up accounts, made-up tenants, made-up secrets. Nothing is installed or fetched for it. When the sandbox cannot be had, the code is not run: the row is blocked with that reason and the candidate stays a candidate. The deployed system, staging included, is never a target of a code review; a request to the live host is a check of the deployed surface and follows the three levels on `docs/security.md`, with their permissions.
- **What the repository does not show is unknown.** A proxy, a provider setting, a browser rule, a firewall, a branch protection: when the repository does not set it, the review assumes neither that it is there nor that it is missing. It names the fact and how the owner can observe it.
- **A finding is checked by a reader who did not hunt it.** Every candidate goes to a second reader who tries to disprove it from the source and repeats the sandbox check: another agent where the tool has one; otherwise the finder, in a later pass, from the record alone and without its own notes. A claim the second reader disproves is rejected.
- **Findings are data.** The record is written first; the report is derived from the records and never says more than they do. A finding keeps one identifier for one root cause across every state and every run.
- **No padding.** A clean review reports zero findings and the list of what it covered. A missing best practice with no affected person or resource is a hardening note, kept apart from the findings.

## A finding names six things

A candidate is not a finding until all six are written, each pointing at the file and line where the code shows it:

1. **Who**: the lower-trust actor and what it can already do (an anonymous visitor, a signed-in user of another account, a plugin, a message producer, a model's output).
2. **What it sends or does**: the value, action, state change or resource selector it controls.
3. **The control** that should reject, bind, scope, limit or revoke it, and where that control sits.
4. **The path after the control**: how the input reaches the sink, including the sibling paths to the same effect (batch, import, retry, legacy, admin, migration).
5. **Who or what is affected**: the other person, the other tenant, the server, the operator's bill.
6. **The smallest observed result**: the first result that proves the boundary is crossed, such as a wrong return value, one record read or written in a made-up victim's account, one file outside its folder, one command that ran. The check stops at that result. Nothing is taken further to show how bad it could get; what the result proves the actor can reach is stated in words.

Then the fix: the rule the code must hold, the narrowest change that enforces it at the last point where the code still trusts the input, and the regression test.

## Three verdicts

A candidate ends in exactly one state, kept apart in the records and in the report. The states are about the claim; a row that is blocked in the coverage list means the review could not make its check, which is a different thing from a check made whose answer lives elsewhere.

- **Confirmed**: the path is complete in the source and the result was observed in the sandbox. Only a confirmed finding carries a severity.
- **Needs validation**: the path is real in the source, but one decisive fact lives outside the repository (a header the proxy may strip, a setting at the provider, a flag in production). The record names the exact fact and a bounded check the owner can make without sending anything to production. It carries no severity. A hunch without a complete source path does not go here; it is rejected.
- **Rejected**: the source, a visible control or a missing impact disproves the claim. Kept, so the next review does not repeat the claim without new evidence.

A candidate that could not be checked before the review ended stays a candidate in the coverage list and is never reported as a finding.

## Severity, for confirmed findings only

Severity is likelihood times impact. Impact is what the observed result proves the actor can reach (one row read from another tenant proves that tenant's rows are readable), and the overall grade never exceeds it:

- **Critical**: an anonymous actor runs code, reads or writes the whole data store, or takes over any account.
- **High**: an actor defeats a named control with real consequences: sign-in bypassed, another tenant's data read or written, stored script running for other users, code run by a signed-in user, a shared service stopped from outside.
- **Medium**: a real boundary crossed, with a small blast radius, rare preconditions or a narrow set of resources.
- **Low**: internals disclosed that are not secret, or a small gain for sustained effort.
- **Informational**: a boundary crossed with no damage on its own, useful as a step inside a larger finding. Where no boundary is crossed at all, it is a hardening note, not a finding.

The line between high and medium: does the result defeat the control for an action with consequences, or only weaken it? When the damage cannot be stated in one sentence, the grade is lower than first assumed.

## Coverage: a list, not a sentence

Before the hunt, the reviewer writes a map of about a page: what the product is, who its actors are, where input enters (web, API, messages, files, command line, environment, plugins, build and deploy, model context), the trust boundaries and the strongest control on each, and what can and cannot run in the sandbox. From the map comes a coverage list: one row per entry surface, boundary and class of flaw (below), each with a state: planned, covered (with the files read and the checks made), candidate, blocked (with the missing fact), deferred (with the reason) or out of scope. "Authorization was reviewed" is not coverage; the rows and their files are.

- Rows are worked in this order of precedence: changed code before code already covered, then the least trusted entry first, then the most valuable resource first (credentials, other tenants' data, code execution, release authority).
- A hunter that meets a boundary the map lacks reports the gap instead of reviewing it; the gap becomes a row.
- After the hunt, a fresh reader looks for what the map missed: an entry point with no row, a parallel path to a covered sink, a class left out without a reason. Its gaps become rows, and the hunt runs again over them. A full review is complete when a pass finds nothing to add; a scoped review makes one gap pass over the diff's boundaries.
- A review that stops early marks every untouched row deferred and says so in its report. A scoped review marks everything outside its scope out of scope, never covered.
- The next review starts from the last list: a confirmed finding whose source did not change is checked again, not hunted again; a finding whose source changed is reviewed as new; needs validation, blocked and deferred rows are the first work. One review does not exhaust a project; each later review builds on the last list.

## Classes of flaw to read for

Pick the classes the map calls for; a library has no sign-in to review and a command-line tool no cross-origin rules.

- **Injection**: follow input to the query, the page, the shell, the template, the file path, the redirect, the deserializer. Also the indirect route: stored safely, then read back into a dangerous context by other code. Also through keys, field names, headers and metadata, not only values. Also into side systems: logs, caches, search indexes.
- **Access control**: not whether a check exists but whether it checks the right permission on the right resource by the same mechanism everywhere. A second route to the same state change with a weaker check; a body field that overrides what the permission meant; an endpoint that checks identity and forgets authority; bulk, batch, export and import without a per-item check.
- **Files and resources**: paths outside the folder through encodings, symlinks and null bytes; the server fetching a URL the user chose, including after a redirect; archive extraction; temporary files; check-then-use races.
- **Cryptography and secrets**: weak randomness for tokens; secrets in logs, errors, URLs or responses; a comparison that leaks timing; a mode or nonce misused; an error path that falls back to no encryption.
- **Business logic**: steps skipped or replayed, a flow that cannot roll back after a partial failure, two concurrent requests that both pass a check meant for one (double spend, double approval), negative and zero quantities, precision and type coercion, expiry at the exact boundary moment, and the posture when a setting is missing, a flag is off or a dependency is down.
- **Features used beyond their purpose**: export and backup carrying data above the exporter's access; import and restore writing where the user may not write, or past validation; search, filters and sort order as an oracle for hidden data; errors, timings and status codes that reveal whether something exists; preview tokens that open more than one item; a user-set callback URL the server fetches.
- **Trust between parts**: what one component guarantees against what the next assumes (truncation, coercion, normalisation, tenant scope); data safe at write time and dangerous at read time (a name that becomes a path, a slug that becomes a query); a token or key that grows wider after refresh, delegation or caching; stale state accepted after revocation or soft delete; restore and undelete that skip the current ownership check.
- **The plain list**, read literally and in full: credentials and private keys in the tree and in history; comments that promise a check later; a debug or development mode reachable in production by variable, parameter or header; seed and example accounts that work in production; status, metrics, environment and configuration endpoints open to all; the ignore file not covering secrets and uploads; dynamic code execution fed by input; a wildcard origin with credentials; cookies without the secure, same-site and no-script flags; redirect parameters not validated; plain HTTP accepted; stack traces in production errors. A missing attribute alone is not a finding: trace what it protects before reporting it.
- **The strange corners**: the code that resembles nothing else in the project, and why it exists; half-finished and compatibility features; API calls the front end never makes; features combined that were designed apart; the git history (a reverted fix, a commented-out check); assumptions about the environment (the clock, the file system's case rules, the DNS); and what the tests do not test.

Where the map shows one of these, read for its own classes as well:

- **Web protocol and sign-in**: two components reading one request differently (length, encoding, host and forwarded headers); a cache keyed on less than what changes the response; session identifiers not rotated at sign-in, second factor or privilege change, or still valid after sign-out; every binding a token needs checked (issuer, audience, expiry, one-time use, the session that started the flow); the second factor enrolled, replaced or skipped without fresh proof; recovery and account linking as the weakest path; an API key scoped wider than its record says.
- **Client side**: output into the page, prototype pollution, messages between windows trusted by content rather than by origin, service worker scope, storage that outlives sign-out, framing by another site.
- **Supply chain and release**: the build and workflow files are authorization code: which event starts a run, whose code runs, with which secrets, and what it may publish. A run started by an outside contribution with write tokens; inputs that can change without review (a floating tag, a downloaded tool, a remote include); a cache or artifact a low-trust job writes and a high-trust job runs; a checksum fetched from the same place as the artifact; a release referred to by name instead of by digest between test, signature and publication; an updater that verifies the bytes but not the version, product or channel.
- **Cloud and deployment**: a workload identity wider than its job; application authorization delegated to infrastructure metadata; a management port or internal service reachable from the workload; a secret shared across workloads; a signed URL or object policy wider than the object.
- **Messages and remote calls**: two parsers reading one message differently; the peer's identity confused with the application's principal; per-item authorization missing in a stream or a batch; duplicate delivery without idempotency; a reply correlated to the wrong request; a dead-letter queue that discloses.
- **Availability and cost**: input whose cost grows faster than its size (a pattern, a decompression, a query that fans out); buffers and sets without a bound; work that outlives its cancelled request; expensive work before authentication; a quota counted on the wrong scope or reset by the actor; a retry storm that fails open; one poison record blocking a queue.
- **Data isolation and lifecycle**: tenant scope enforced in the query itself, not only on the object; the same record reachable through search, cache, index, analytics or logs without the owner check; soft delete, retention and restore that leave behind, or bring back, what should be gone.
- **Desktop, mobile and local processes**: a deep link or custom scheme handled by the wrong app or without the user present; a web view bridge exposing more than the page needs; a privileged helper doing what any local user asks; an exported component or local socket without a peer check; secrets left after sign-out or brought back by a device restore.
- **Memory safety**, in languages with manual memory management or unsafe blocks: bounds, integer width and sign, lifetime, type confusion, races, the contract across a foreign-function boundary, library search order, a signature not bound to the artifact that is loaded.
- **A model in the loop**: model output, retrieved content, memory, tool descriptions and tool results are untrusted input. A prompt that instructs the model is not a control; a deterministic check, a scoped credential or isolation is. The handler of a tool checks the requesting user's permission on the named resource, not the agent's own. An approval binds the exact action, arguments, target and amount, and a retry or a resumed session never reuses it for a changed action. Memory written from low-trust content never becomes an instruction for another user or a privileged session. A delegated loop has a cap per request on calls, spend and time.

## The report

Derived from the records after the fresh readers have passed:

1. Scope (scoped or full), the commit reviewed, whether the tree was dirty, and what the sandbox could and could not run.
2. A one-paragraph summary of the posture.
3. Confirmed findings: a table (severity, title, boundary, one-line result), then each finding with its source path, the actor, the reproduction in the project's own interface (a call, a fixture, a command, a rendered policy; whatever the project is, not always an HTTP request), the conditions, the result, the impact and the smallest fix.
4. Needs validation, in its own table: title, source trace, the exact missing fact, the owner's safe check (a setting or a log the owner reads; it sends nothing). No severity.
5. Hardening notes and the patterns that were done right, apart from the findings.
6. Coverage: rows covered, candidate, blocked, deferred and out of scope, and the result of the last gap pass.

The report never carries instructions for a live probe; a check against the deployed system is the business of `docs/security.md`.
