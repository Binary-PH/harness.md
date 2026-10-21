# Playbook: motion graphics and video

A short video made from code: an explainer, a product teaser, a simulation, a 3D flythrough. The piece is built from a written timeline, drawn frame by frame, judged from its own frames, and given sound from the same timeline.

## 1. Agree the piece before any code

- Agree the message, a storyboard and one hero moment with the user. One hero moment per piece, built toward, never spent early.
- Fix the format (1:1, 16:9 or 9:16), the length, the frame rate (60 by default) and the brand colours and fonts when they matter.
- Demo content uses invented people, invented brands and example domains, never a real person or company.

## 2. One timeline is the source of truth

- Every beat is a named timestamp in seconds, in one timeline file per piece, grouped by scene. Scenes, sound and review all read that file. A start time is never typed into a component or a cue sheet; an offset from a beat is fine.
- Retiming the timeline retimes everything. The sound is generated again after any retime.
- The palette and the fonts live in one place. A component never adds a colour or a font of its own.

## 3. Every frame from the time alone

- A frame is computed from the frame number or the time and nothing else: no CSS animation, no clock, no unseeded random value. Seed every random value so a rebuild gives the same frames.
- With that rule any frame can be drawn on demand, which is what makes the review loop and frame-exact capture possible.

## 4. Two pipelines

- **Motion graphics**: a code-to-video framework where each scene is a composition drawn from the current frame. Shared pieces (a masked text reveal, a rolling number, grain, a drifting background) live in a components folder and are reused.
- **Simulations and 3D**: one self-contained page per piece, with the 3D library and the fonts inlined so it runs offline. The heavy work (agents, paths, collisions) is simulated once at load; a draw function takes a time and draws that moment. A headless browser steps that function frame by frame and pipes the frames to the encoder, so the output is frame-exact at the chosen size and rate however slow a single frame is to draw. The same page plays live in a browser for a quick look.
- Capture needs a browser binary on the machine. Install it once, from the setup script.

## 5. Motion defaults, with numbers

- Easing: entrances ease out, exits ease in, a move between two states eases in and out, a value that lands gets a slight overshoot. Linear only for continuous motion (rotation, drift, grain).
- Exits take about half as long as entrances (0.35 to 0.55 s against 0.7 to 1.25 s). Scenes overlap: the next starts before the previous has left, because a hard gap reads as loading.
- Springs for physical objects (a settle: damping 14, stiffness 140, mass 0.8; a pop: damping 9 to 11, stiffness 200 to 230; one elastic accent per piece at most). Curves for type and camera moves.
- Stagger: about 0.09 s between words, 0.2 to 0.25 s between list items, about 0.06 s between dots or rings. A word reveal takes about 0.8 s. Typing runs about 0.035 s per character.
- Reading holds: at least 1 s for a short phrase plus about 0.25 s per word beyond three, and at least 2 s for the final line. When unsure, hold longer and animate less.
- Type: at most three families (a display face, a text face, a monospace for labels and numbers). Display type about 138 px on a 1080 px canvas. One accent colour, used only to mark what changed, the answer or the focal point. The key message stays readable at 360 px wide.
- Texture: a light film grain (about 0.09 opacity, reseeded from the frame number every two frames), a vignette, a slowly drifting grid. Idle elements keep moving a little.
- Continuity: objects persist across scenes and change role instead of being cut away. Every transition is caused by something on screen; no generic wipes. One focal point per frame. Important content stays about 60 px from the edge. Frame 0 is a finished image, a held opening state before the first entrance, because autoplay thumbnails use it.
- Shaders: clamp the base of every power function. One NaN pixel can black out a whole frame through bloom.

## 6. The review loop

The agent judges frames, since it cannot watch the video play. A piece is reported done only after the frames of its latest render have been read.

1. Render a half-resolution draft.
2. Build a contact sheet: one frame per beat of the timeline, taken about 0.3 s after the beat so the motion has settled, tiled with the timestamp on each frame.
3. Open every sheet and go through the list: one focal point per frame; no unintended overlap, above all during transitions; no clipped type; the key message readable at phone width; text still while it is meant to be read; holds long enough; exits faster than entrances; only palette colours; recurring elements the same in every scene; frame 0 not blank.
4. Look closer at anything doubtful: scrub a transition with a frame every 0.2 s, or pull one full-resolution frame.
5. Fix, render again, back to step 1. Stop when a full pass finds nothing.
6. When a pass is clean, render at full resolution and add the sound (section 7). Then a last pass on the finished file: the sheet again, and the waveform check. A finding here goes back to step 5.

Report what was checked, what was fixed and which sheet shows each fix. A fix that has not been rendered again and read is not a fix.

## 7. Sound from the same timeline

- Sound is generated in code (short synthesised hits, risers, chimes, a pad), so there are no samples to license and the output is identical on every run. One cue sheet per piece reads the timeline and places each sound at a beat.
- Events and sounds: typing gets keystrokes; a reveal gets a whoosh and a soft landing; a pop-in gets a pop; a snap gets a click; one riser per piece for the build-up; a success moment gets a chime; the hero transition gets a whoosh and one impact; a pad sits under everything. Continuous motion stays silent. A sound marks a moment; not every movement gets one.
- Levels: hero hits 0.25 to 0.35, supporting sounds 0.08 to 0.2, the pad about 0.03, lowered by about two thirds under the hero transition. Pan follows the position on screen. Tonal sounds share one key, rising as the piece progresses.
- Master to -14 LUFS, within 1, with the true peak at or below -2 dBFS. A mix of short hits and silence that lands quiet is fixed with sustained sound (a pad, longer tails), never by forcing the gain.
- Verify with a waveform drawn with the beat markers: every transient sits on or just after a beat line. A hit between beats means a typed time or a stale sound file.
- Voice, when needed, comes from a generated file placed on the timeline like any other sound.
- Export: mux the picture and the mastered sound, with the stream index at the front of the file for playback over the web, then measure the loudness of the final file.

## 8. Music-driven pieces

When the piece runs to a track, detect its tempo, bars and drop first, place the clip so the drop lands about 40 percent in, and snap the major moments to the beat grid.

## 9. Keep the gallery

Each finished piece gets an entry in an examples folder: the prompt it came from, the process notes, and the final file or a link to where it is published. The next piece then starts from a real reference.

## Checks and tests to add

- The gate's linter and type check cover the composition and simulation code; the timeline file is validated against its schema.
- A test renders the contact sheet headless and fails when a frame is blank or a beat has no frame; a loudness check fails when the final file is outside the target.
- Drafts, contact sheets, sound stems and intermediate renders are output, not source: they go to the folder the project names for runs, stay out of git, and are deleted when the piece ships. Only the final file is kept, where the gallery entry points.
