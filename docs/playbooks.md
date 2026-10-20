# Playbooks: decide before coding

What to do before the first line of code, by kind of project. The gate, the tests and the ship flow apply to every kind; this page only decides the approach.

## The order of decisions

1. **Understand the ask.** Restate it in one paragraph. Note what is fixed (language, platform, host) and what is open.
2. **Research first.** Find open-source projects that do what the user wants, on code hosting sites and in package registries. Read their structure, their issues and their pitfalls. Write down, in your own words, what to take (layout, data model, tool choices) and what to avoid. Respect their licenses: ideas are free; copy no code.
3. **Choose the stack.** Build with a modern, maintained language and framework unless the user names one. "Modern" means current major versions, typed where the language allows, with an active ecosystem. Say the choice and the reason in one sentence.
4. **Install or use the best tools for the job.** Start from the tools table (`docs/tools-by-language.md`); take an equivalent that fits the project better when one exists and say why. Download and install only tools compatible with the chosen language and platform. A Python project gets no JavaScript tool chain, a Flutter app no PHP tooling. Pin versions in a setup script so a drift shows in the preflight.
5. **Set up the gate and the runner** (`docs/code-check.md`, `docs/testing.md`) before the first feature, not after.
6. **Then build**, one finished piece at a time, each checked and tested.

## WordPress sites and plugins

- Add `phpcs` with the WordPress coding standards and the PHP compatibility standard to the gate. Errors block, warnings are reported.
- Escape on output, sanitise on input, a nonce on every action, a capability check on every privileged action, prepared statements for every query.
- Every plugin or theme zip gets a new version (the third, second or first number by the size of the change) and the version, or a content hash, in the file name of every asset it serves, so no cache serves a stale copy.
- Keep the static security analysis and secrets scan from the gate; they cover PHP as well.

## 3D renders

When the ask is a 3D model or scene built from a reference image, follow `docs/playbooks/3d-renders.md`: study the image, write a spec, build in passes, judge against the reference, keep a triangle budget.

## Interactive web pages

When the ask is a scroll-driven, animated or 3D web page, follow `docs/playbooks/interactive-web.md`: hero first, inspect supplied models, tie the camera to scroll, keep motion accessible and the page light.

## Anything else

Research how it is done first (step 2 above), write the approach in a few lines, then apply the same gate, tests and ship flow. Do not invent a method when a proven one exists.
