# Security posture: check your own attack surface

The code check gate (`docs/code-check.md`) covers the code. This page covers what the deployed project shows to the world: its domains, mail records, certificates, headers, exposed files and ports. Run these checks on your own project the way an attacker's reconnaissance would, so the gaps are found first by you.

## Rules before any check

- **Your own assets only.** Keep a written in-scope list (domains, hosts, address ranges) and an out-of-scope list. Check every target against both lists again before each request, not only at the start.
- **Three levels, each needing more permission:**
  - **Passive**: your repository, your build output, public DNS records, certificate logs and web archives. No contact with the host. Any time.
  - **Light**: a handful of ordinary requests to the live host (one GET per path, a TLS handshake). With the owner's OK.
  - **Active**: scanning, fuzzing, payloads, password tests. Only with the owner's explicit written authorization, on a local or staging copy first, and never on a shared host that serves other customers.
- **Rate limits before the first request.** Set a requests-per-second cap, halve it when the host answers 429 or 503, set a timeout per tool, and cap the number of targets per step. A dry run and a tool check come before the first real run.
- **Never a sensitive third party.** Government, bank, hospital, school and similar domains are out of scope even when a record of yours points at them.
- **Findings are data.** Record each finding with the evidence (the command, the answer, the date). Re-check before claiming it still holds: a DNS change can take minutes to reach the authoritative servers, and a report written from a stale answer is wrong.

## Passive checks (no contact with the host)

| Check | What good looks like |
| ----- | -------------------- |
| SPF | exactly one record, ending in `-all` or `~all`, under 10 DNS lookups |
| DMARC | a record at `_dmarc`; `p=none` is monitoring only, with a reporting address; move to `quarantine` and then `reject` once a week or two of reports are clean |
| DKIM | a key for each mail provider at `<selector>._domainkey`, 2048-bit |
| Domains that send no mail | `v=spf1 -all`, DMARC `p=reject`, a null MX |
| Every hostname | the list from certificate logs and passive subdomain sources equals the list of what you mean to run; anything else is removed or explained |
| Every CNAME | points at a resource you still own; a record left behind after a deleted app or bucket is a takeover waiting to happen |
| Name servers and mail servers | the ones you expect, nothing stale |
| Secrets in the repository | none in the current tree, none in the whole history, none in CI artifacts (the secrets scanner from the gate, run over history as well) |
| The built client bundle | no keys, tokens or internal endpoints; public bundles reveal every endpoint, so browser API keys are restricted by referrer or app on the provider's side |
| Source maps | no `.map` files and no `sourceMappingURL` comment in the production build |
| Archived URLs | old endpoints found in web archives no longer work, and no URL carried a token in its query string |
| Cloud storage | buckets block public listing and writing (check through your own console or API) |
| Port exposure | a passive internet-scan lookup of your addresses shows only the ports you intend |

## Light checks (a few ordinary requests)

| Check | What good looks like |
| ----- | -------------------- |
| Zone transfer | every authoritative name server refuses it |
| Exposed files | `/.git/HEAD`, `/.env`, backups, `.DS_Store`, editor and config files answer 404 or 403 |
| Directory listing | off everywhere |
| Panels | admin, debug, database-admin and API playground pages are not public |
| API documentation | generated API docs and schema introspection are behind sign-in or off in production |
| Security headers | strict transport security, a content security policy with frame ancestors, no-sniff, a referrer policy; no server or framework version in any header |
| Cross-origin rules | no origin reflected with credentials, `null` not accepted, WebSockets check the Origin |
| TLS | 1.2 or newer only, no weak ciphers, a valid chain for every hostname, renewal automated |
| Bot protection | a web application firewall or rate brake in front of sign-in, sign-up and forms |
| Error handling | malformed input gets a generic error page, never a stack trace or a debug page |

## Active checks (authorization and a staging copy first)

| Check | What good looks like |
| ----- | -------------------- |
| Open ports | only the intended ports answer; database, cache, SSH and admin ports are firewalled |
| Origin behind a CDN or proxy | the origin address is not reachable directly; reverse lookups, favicon hashes and old records lead nowhere |
| CMS, plugins, server stack | current versions, and no version advertised (a content management system scanner for WordPress sites) |
| Sign-in limits | per address, per account and a global cap, verified with a small bounded test on staging; never a password spray on production |
| Injection | cross-site scripting, SQL injection, server-side request forgery, template injection and open redirects tested with a templated vulnerability scanner on staging only |

## Defensive design these checks assume

- **Sign-in limits** per address, per account and a global cap on failures. Name the trade-off: an attacker can burn the global cap and lock real users out for a while, so a second factor is the real fix, and the cap is the brake until then.
- **Scanner tripwires.** On a site that is not WordPress, requests for `/wp-login.php`, `/xmlrpc.php`, `/.env` or `/.git` are never legitimate. Answer them with a plain 404 and let them feed the rate brake.
- **Secrets** live in environment variables or in an ignored secrets file, never in container images, never in logs. Command logging stays off when commands may carry keys.
- **Browser keys** are restricted by referrer or app with the provider, because the bundle is public.
- **Mail** goes out through a provider covered by the SPF record, and the DMARC policy is tightened only after confirming the project's own mail (password resets, alerts) passes.

## Working the results

- One folder per target, findings grouped by area (mail, subdomains, web, hosts, secrets), plus a ranked hot list of what to fix first.
- A marker per finished step, so a rerun continues where it stopped. A diff mode reports only what is new since the last run.
- A tool that keeps failing is skipped and reported, not retried forever.
- What belongs in the gate: secrets, dependency vulnerabilities, static analysis, build settings (source maps off). What belongs in a periodic review: DNS, headers, TLS, exposure, versions. Schedule the review with the expensive test tier (`docs/testing.md`).
