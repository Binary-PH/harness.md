# Playbook: a 3D render from a reference image

Rebuild the object in a reference image as a code-built, animation-ready 3D model that a browser renders. The model is generated from a written spec, judged against the reference, and kept within a triangle budget.

## 1. Study the image before any code

- Name the object. Split it into parts from large to small. Describe each part's material in physical terms (metal or not, rough or glossy, clear coat, transmission).
- Classify the subject (object, character or mixed) and its complexity.
- Write the must-match list: the features without which it is not this object.
- List the small identity details (bevels, seams, fasteners, engraved lines, gloss against matte, wear) and tie each one to a part or a material. Drop any detail you cannot place.
- Write down what one view cannot show, and mark those regions as guessed.

## 2. Keep a written spec as the source of truth

The spec holds the part tree, the materials, the lights, the pivots and the attachment points. Code is generated from the spec; a change goes into the spec first, then the code is regenerated. A spec that is too shallow (few parts, no identity details, no lighting) blocks code generation until it is deepened.

## 3. Build from code, not downloaded meshes

- Primitives, extrusions, lathes, tube and tapered sweeps; instanced meshes for repeated parts (screws, vents, buttons); carving from two or more silhouettes when the shape needs it.
- Every random value is seeded so a rebuild gives the same model.
- The result is one group with named, pickable meshes and pivots, so parts can be animated, exploded or highlighted later.

## 4. Build in ordered passes

Blockout, structure, form, material, surface, lighting, interaction, optimisation. Each pass touches only its own concern. Record one decision per pass: continue, fix the spec, fix the code, ask for input, or stop. Cap corrections at three per pass and six in total; past that, report what does not match and why.

## 5. Materials

- Physically based materials with values per finish type (metalness, roughness, clear coat, transmission, index of refraction, environment intensity, anisotropy).
- Colours come from the reference pixels, not from a written brief. When they disagree, follow the image and say so, unless the user asked for the change in words; the user's words win.
- Colour maps in sRGB; roughness, normal and occlusion maps linear and in separate channels.
- Procedural maps at least 1024 pixels, 2048 for a hero shot. A patterned finish uses the reference crop with its baked lighting removed; flat paint gets a solid colour.
- Candy or anodised coats: some metalness plus a clear coat, so the colour survives the reflections.

## 6. Lighting

The spec names the key light (direction, colour temperature, intensity, softness), the fill, the rim or the environment, the ambient level, the exposure and tone mapping, the background and a soft contact shadow. Review the model under three setups: neutral, low-angle grazing, and matched to the reference. A model that only works in one light is not finished.

## 7. Capture the same way every time

- A fixed viewport (for example 1600 by 900), orbit controls off during capture.
- Wait until every texture has loaded before the screenshot.
- One "match" view framed like the reference, with the object filling about the same width.
- A turntable at 0, 90, 180 and 270 degrees.

## 8. Judge against the reference

Put the reference and the render side by side in one image and judge that image. Score silhouette, structure, form, material and lighting separately. A pass fails when any critical feature is wrong, even if the average looks fine. Geometry checks (bounds, symmetry, part counts) run alongside the visual judgement.

## 9. Budget and performance

- Set the triangle budget up front: about 6,000 for a low tier, about 60,000 for standard, more only for a hero shot. Decimate per part when over budget, in the optimisation pass.
- Record draw calls and triangle counts from the renderer after each pass.
- Optimise only after the look is approved; a fast wrong model is still wrong.

## 10. Honesty rules

- Depth, segmentation or landmark models may give hints (relative depth, masks, key points). They never produce the geometry and never approve a pass.
- No visual claim without a saved screenshot that you opened and read back. Name the regions that are guessed. "Not reachable from this image" is an acceptable answer.

## Checks and tests to add

- The gate's linter and type check cover the generated code; the spec file is validated against its schema.
- A test renders the model headless, compares bounds and part names with the spec, and fails when the triangle count exceeds the budget after the optimisation pass.
