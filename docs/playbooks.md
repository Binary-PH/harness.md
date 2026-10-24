# Playbooks: what a kind of project adds

What a kind of project needs before and while it is built. Planning itself (the ask, the research, the documentation of the version in use, the stack, the tools, the plan and its slices) is the same for every kind and lives on `docs/planning.md`; the gate, the tests and the ship flow apply to every kind as well.

## WordPress sites and plugins

- Add `phpcs` with the WordPress coding standards and the PHP compatibility standard to the gate. Errors block, warnings are reported.
- Escape on output, sanitise on input, a nonce on every action, a capability check on every privileged action, prepared statements for every query.
- Every plugin or theme zip gets a new version (the third, second or first number by the size of the change) and the version, or a content hash, in the file name of every asset it serves, so no cache serves a stale copy.
- Keep the static security analysis and secrets scan from the gate; they cover PHP as well.

## 3D renders

When the ask is a 3D model or scene built from a reference image, follow `docs/playbooks/3d-renders.md`: study the image, write a spec, build in passes, judge against the reference, keep a triangle budget.

## Interactive web pages

When the ask is a scroll-driven, animated or 3D web page, follow `docs/playbooks/interactive-web.md`: hero first, inspect supplied models, tie the camera to scroll, keep motion accessible and the page light.

## Motion graphics and video

When the ask is a short video, an explainer, a simulation or a 3D flythrough made from code, follow `docs/playbooks/motion-video.md`: agree the storyboard and one hero moment, write one timeline of named beats, compute every frame from the time alone, review contact sheets of the frames until a pass is clean, add generated sound from the same timeline.

## Web apps and sites the public reaches through search

When the project is a web app, a site or any page a search engine or an answer engine can reach (a marketing site, a shop, a blog, documentation, a local business page), follow `docs/playbooks/web-seo.md` as well: decide the kind of site, serve what matters in the first HTML, keep crawling and indexing clean, meet the speed and image targets, write pages with an author, dates and a direct answer, add truthful structured data, and audit with findings that say how they would fail. Internal tools, APIs, pages behind sign-in and native apps skip it.

## Anything else

Research how it is done first (`docs/planning.md`, the order of decisions), write the approach into the plan, then apply the same gate, tests and ship flow. Do not invent a method when a proven one exists.
