# Playbook: search visibility for a web app or site

Applies to anything the public reaches through a search engine or an AI answer engine: a marketing site, a shop, a blog, documentation, a local business page, the public pages of a web app. It does not apply to internal tools, APIs, pages behind sign-in or native apps. The gate, the tests and the ship flow still apply; this page adds what search needs.

## 1. Decide the kind of site first

- Read the home page for signals and name the kind: a product with pricing, features and docs; a local service with a phone number, an address and a map; a shop with products and a cart; a publisher with articles and authors; an agency with case studies. The kind selects the extra checks in section 8. Two kinds at once: show the evidence for both and ask the owner.
- Decide with the owner which crawlers may read the site, by exact user-agent name, before the first page ships. A crawler that trains a model and a crawler that answers searches are separate decisions; an answer engine that is blocked cannot cite the site. Fetchers triggered by a user often ignore the robots file, so anything that must stay private sits behind server-side access control, not behind a robots rule.

## 2. Serve what matters in the first HTML

- The title, the description, the canonical, the robots meta tag, the structured data and the primary content are in the HTML the server sends. A client-rendered shell shows them only after scripts run, and many crawlers and most answer engines do not run scripts. Public pages are rendered on the server or at build time; client rendering is for pages behind sign-in.
- The raw HTML and the rendered page say the same thing about the canonical and about indexing. When they disagree, either may win. A noindex in the raw HTML can hold even if a script removes it.
- Key content and structured data sit within the first 2 MB of the HTML.
- A separate copy of the page served to bots is debt, not a setup.

## 3. Crawl and index

- A robots file exists, is valid and blocks no resource a page needs to render. A server error on the robots file reads as "disallow everything".
- The XML sitemap lists only canonical pages that answer 200 and are indexable, with honest last-modified dates; it is named in the robots file; past 50,000 URLs or 50 MB it is split under an index file.
- Every indexable page carries one self-referencing canonical with an absolute URL, never together with noindex. After a canonical fix, the engine may keep the old grouping for about two weeks; an unchanged result right after the fix is not a failed fix.
- Noindex is intentional only: keep the list of what is excluded and why.
- One address per page: the www and bare host, http and https, and trailing-slash variants each redirect once to the canonical. A redirect chain is at most one hop, and a permanent move is a 301.
- Important pages sit within three clicks of the home page. No orphan pages. Paginated lists use real links, a self-canonical per page, and a URL for every page that a load-more button would reveal.
- A multi-language or multi-region site marks every variant with reciprocal, self-referencing language annotations, an explicit default, absolute URLs and one language per URL.
- Crawl budget matters past about 10,000 pages; then index bloat, parameter URLs and faceted pages get attention.

## 4. URLs, security and mobile

- URLs are descriptive, lowercase and hyphenated, carry no parameters for content, use the trailing slash consistently, stay under about 100 characters, and mirror the site's structure in their folders.
- HTTPS everywhere with a valid chain and no mixed content, plus the headers from `docs/security.md`. A page that fights the back button through history manipulation is a critical finding.
- A viewport tag, a responsive layout, no horizontal scroll, touch targets of at least 24 by 24 CSS pixels, a base font of about 16 pixels.
- Parity between the mobile and desktop versions: the same primary content, the same robots tag, the same title and description, the same structured data. Primary content is never behind a tap, and never lazy-loaded in a way that needs an interaction to appear; native lazy loading of images below the fold is fine. No full-page interstitial; a small banner or a legal dialog is fine.

## 5. Speed

- The three field metrics at the 75th percentile of real users, mobile and desktop measured apart: largest contentful paint at or under 2.5 s, interaction to next paint at or under 200 ms, cumulative layout shift at or under 0.1. Field data first; for the paint and layout metrics, lab data stands in when the site has too little traffic, with a retest once it grows; the interaction metric needs real users and has no lab stand-in. The retired input-delay metric is never cited.
- Split the largest paint into server time, resource delay, load time and render delay to see where it goes.
- Images: the hero image is fetched with high priority and never lazy-loaded; everything below the fold is lazy-loaded; every image has width and height or an aspect ratio; modern formats with a fallback; a responsive source set at about 400, 800 and 1200 pixels wide; a descriptive, lowercase, hyphenated file name; alt text of 10 to 125 characters that says what the image shows, empty only on decoration. Size targets: a thumbnail under 50 KB, a content image under 100 KB, a hero under 200 KB.
- Assets carry their version in the file name (section 2 of `HARNESS.md`) and are served with long cache lifetimes; an image-heavy site serves them from a CDN.

## 6. Page content

- One H1, then a clean H2 and H3 hierarchy with no skipped level. Headings are scannable; where people search with a question, the heading is the question.
- Each page has a unique title and description. The primary topic appears in the title, the H1 and the first 100 words, and nowhere by force.
- Coverage floors by page kind, not targets: a home page 500 words, a service page 800, an article 1,500, a product page 300 (400 for a complex product), a location page 500 to 600.
- Paragraphs of 2 to 4 sentences, sentences of 15 to 20 words on average, a table for a comparison, a list for steps, a table of contents on a long page. Aim for a reading-ease score around 60 to 70 for a general audience.
- Internal links: 3 to 5 per 1,000 words with descriptive anchor text. External links go to primary sources.
- Who, how and why: a visible author with credentials (required for health, money and safety topics), a publication date and a last-updated date, a purpose that helps the reader rather than fills a query, and original material (data, cases, first-hand photos). Content on a moving topic that is more than 12 months old without an update is flagged.
- Trust pages weigh most: contact details, an address, a privacy policy, terms, a corrections note.
- The risk is scaled low-value text (generic, repetitive across pages, unattributed, inaccurate), not machine assistance itself. Before publishing a draft, strip invisible characters and stock phrasing from it, and do that only when the owner owns the content.
- For answer engines: the direct answer in the first 40 to 60 words of a section; self-contained passages of about 130 to 170 words (a reading heuristic, not an engine rule); specific, attributable facts with numbers; a plain definition in the first 60 words of a page about a thing; freshness and a byline. The brand exists outside its own site: an encyclopedia entry, forum mentions, video, professional profiles. A model-oriented text file at the site root is harmless and earns nothing; chunking and rewrites in an "AI style" earn nothing.

## 7. Structured data

- JSON-LD, with absolute URLs and only truthful data, validated before it ships. Anything time-sensitive (a product, an offer) goes in the initial HTML.
- Types that still earn rich results: organisation, local business, product and offer, article and blog posting, breadcrumb list, web site, person and profile page, video, event, job posting, course (the course type itself), software application, review and aggregate rating. A review is never invented, and an incentivised review says so on the page. Question-and-answer markup only for genuine user Q&A.
- Never add: how-to, special announcement, claim review for search features, vehicle listing, estimated salary, learning video, the course-info carousel, practice problem. FAQ markup earns no rich result anymore: leave existing markup alone and add none for ranking.
- A block without a context or a type is flagged.

## 8. By kind of site

- **Local**: the name, address and phone number identical everywhere they appear; the business profile complete with current fields; reviews answered. One location page per real location; past 30 location pages, each needs at least 60 percent unique content; past 50, stop until the owner justifies them.
- **Shop**: product markup with offer, price and availability in the first HTML; faceted navigation kept out of the index (noindex on filtered pages, or a canonical to the unfiltered page); a generated product image carries the source-type metadata the merchant platform requires.
- **Publisher**: article markup, author pages, topic clusters around pillar pages, visible dates.
- **Product**: pricing, features, docs and integrations pages indexable, with the docs rendered on the server. A generated API reference and schema introspection stay behind sign-in (`docs/security.md`).
- **Programmatic pages** (many pages from one template): each page holds unique value or it is not indexed; thin ones are merged or marked noindex.

## 9. The audit and the report

- Measure only what can be measured. "Not measured" is a status, never a pass, and a score covers only measured categories, with the checks behind it shown.
- Each finding is critical (blocks indexing or risks a penalty: fix now), high (within a week), medium (within a month) or low (backlog).
- Each recommendation carries four things: the observation it rests on, what it depends on or unblocks, how we would know it failed, and a leading indicator the owner can watch without a new audit.
- Keep a baseline snapshot (index count, field metrics, rankings for the key queries) and compare after each release.
- Before blaming a traffic change on an engine update: a date overlap is a hypothesis, never proof. Check the engine's status page and the site's own changes first.
- Credentials for a search console or analytics API live in an ignored config file readable by the owner only, never in the repository.
- An unreachable URL is reported with its status; its content is never guessed.

## Checks and tests to add

- To the gate, for a web project: a build-time check that every public page has a unique title, a description, one H1, a self-canonical, an alt attribute (empty only on decoration) and dimensions on every image, and valid JSON-LD.
- A test that fetches the raw HTML of the key pages headless and asserts the title, the description, the canonical, the robots tag, the structured data and the primary content are present before any script runs.
- A sitemap test: every listed URL answers 200, is canonical and is indexable; the robots file names the sitemap. It runs against the staging copy, or against production at a capped rate.
- On a site whose content is written in Markdown, the test filters map the content folder to the raw-HTML and sitemap tests, so a content change is not a docs-only change.
- A lab run of the paint and layout metrics on the key pages in the expensive tier (`docs/testing.md`), failing at the thresholds in section 5, with the interaction metric read from field data; the periodic security review (`docs/security.md`) adds the robots, sitemap and redirect checks.
