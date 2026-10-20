# Playbook: an interactive web page

Scroll-driven, animated or 3D web pages: a hero with a model or a scene, sections that move with the scroll, galleries with hover and motion. Build it one section at a time and keep it usable for everyone.

## 1. Hero first

- Build only the hero until it is approved. Its look, type, motion and colours become the style guide for every later section.
- When a reference video or screen recording is supplied, match its motion and interaction, then apply the user's explicit changes on top (closer opening shot, headline behind the subject, and so on).
- Set the opening framing explicitly: distance, angle, what is in view before any scroll.

## 2. Inspect a supplied 3D model before using it

- Read its header: animation clips, skeleton and joints, triangle count, mesh count, compression and texture formats. Load the matching decoders (mesh compression, quantisation, modern image formats).
- A model with a skeleton but no clips needs a seamless looping animation written on the bones (a walk, an idle). Review it limb by limb: legs and feet are where a generated walk goes wrong first.
- Note the weight. A compressed hero model can still be several megabytes and hundreds of thousands of triangles. Decide early whether it ships as is, decimated, or replaced by a lighter copy on phones.

## 3. Motion rules

- Tie the camera path to the scroll position, orbiting or tracking the subject. Scroll drives motion; motion never hijacks scroll.
- Keep the headline behind the subject so text never covers it.
- Add effects (an aura, glow, particles) only after the base motion is right.
- Honour the reduced-motion preference: when it is set, replace scroll-driven motion with static frames or short fades, and never auto-play looping video.

## 4. Galleries and sections

- A card gallery can sit on a spiral around a central object: scrolling rotates the spiral and moves it down in perspective; cards carry a glass look and a hover state.
- Looping video on only a few cards, each loop short (three to five seconds), muted, lazy-loaded.
- Generate all media to one art direction (the same character, the same worlds), so the page reads as one piece.
- A footer and every later section reuse the hero's spacing, type and colour tokens.

## 5. Accessibility

- Every control reachable by keyboard, with a visible focus state. Every image and model has a text alternative that says what it shows.
- Screen readers get the content in reading order, independent of the visual motion.
- Contrast holds in both light and dark appearance.

## 6. Performance budget

- Set a budget before building: first render time, total transfer on a phone connection, and no visible jank while scrolling.
- Lazy-load everything below the fold: models, videos, large images. Preload only what the hero needs.
- Test at phone width with a 16 pixel side gutter and no horizontal scroll; test with the device's reduced-motion setting on.
- Measure, do not guess: take the numbers from the browser's performance tools and keep them with the page.

## 7. Iterate with observable feedback

Give and ask for feedback that names what is seen: which body part, which shot, which layer, which second of the scroll. Fix that, re-capture, compare. Do not start over when a specific fix will do.

## Checks and tests to add

- The gate's linter, type check and static analysis cover the page code; the secrets scan covers the built bundle.
- A smoke test loads the page headless, scrolls to each section, checks that the expected elements are visible, and fails when the transfer size or the model's triangle count exceeds the budget.
