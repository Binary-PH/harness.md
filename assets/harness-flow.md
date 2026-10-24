# Source of the loop diagram

`harness-flow-light.png` and `harness-flow-dark.png`, the diagram in the README, are rendered from the page in the code block below. The page is the source; the two images are its output, and nobody edits them by hand. A change to the diagram edits this page and renders both images again in the same commit:

1. Save the code block as an HTML file in a scratch folder outside the repository.
2. Open it in a headless browser and take a screenshot of a window 1200 by 600 pixels, at a device scale factor of 2, with a transparent background. The page as it is gives the light version; the same page with `#dark` at the end of its address gives the dark one.
3. Reduce each screenshot to a palette of 256 colours, keeping its transparency, and save it over the PNG of the same theme.
4. Look at both images, and when a word in a card changes, change the alt text of the diagram in the README to match.

The page uses only generic font families and has no script, so it follows the rules of the text like every file here.

```html
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<style>
  :root {
    --text: #1f2328; --muted: #59636e; --card: #f6f8fa; --border: #d0d7de;
    --accent: #0969da; --on-accent: #ffffff; --line: #8c959f;
  }
  body:target {
    --text: #e6edf3; --muted: #9198a1; --card: #161b22; --border: #30363d;
    --accent: #4493f8; --on-accent: #0d1117; --line: #6e7681;
  }
  html, body { margin: 0; padding: 0; background: transparent; }
  body { width: 1200px; height: 600px; position: relative; overflow: hidden;
         font-family: system-ui, sans-serif; color: var(--text); }
  .card { position: absolute; width: 255px; height: 178px; box-sizing: border-box;
          background: var(--card); border: 1.5px solid var(--border); border-radius: 14px;
          padding: 16px 18px; }
  .head { display: flex; align-items: center; gap: 10px; }
  .num { width: 30px; height: 30px; border-radius: 50%; background: var(--accent);
         color: var(--on-accent); font-weight: 700; font-size: 16px; display: flex;
         align-items: center; justify-content: center; flex: none; }
  .title { font-size: 25px; font-weight: 700; letter-spacing: 0.2px; }
  .file { font-family: ui-monospace, monospace; font-size: 13.5px; color: var(--accent);
          margin: 8px 0 8px; white-space: nowrap; }
  .line { font-size: 16.5px; line-height: 1.32; color: var(--text); }
  .line.muted { color: var(--muted); }
  svg { position: absolute; left: 0; top: 0; }
  .label { position: absolute; font-size: 15px; color: var(--muted); font-style: italic; }
</style>
</head>
<body id="dark">
<svg width="1200" height="600" viewBox="0 0 1200 600">
  <defs>
    <marker id="a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="var(--line)"/>
    </marker>
  </defs>
  <g stroke="var(--line)" stroke-width="2.2" fill="none" marker-end="url(#a)">
    <line x1="281" y1="114" x2="320" y2="114"/>
    <line x1="581" y1="114" x2="620" y2="114"/>
    <line x1="881" y1="114" x2="920" y2="114"/>
    <line x1="1051" y1="205" x2="1051" y2="390"/>
    <line x1="922" y1="481" x2="883" y2="481"/>
    <line x1="622" y1="481" x2="583" y2="481"/>
    <line x1="322" y1="481" x2="283" y2="481"/>
  </g>
  <path d="M151,392 C151,300 451,300 451,207" stroke="var(--line)" stroke-width="2.2"
        stroke-dasharray="7 6" fill="none" marker-end="url(#a)"/>
</svg>
<div class="label" style="left: 322px; top: 316px;">next task</div>

<div class="card" style="left:24px; top:25px;">
  <div class="head"><div class="num">1</div><div class="title">Read</div></div>
  <div class="file">HARNESS.md</div>
  <div class="line">Whole, once a session</div>
  <div class="line muted">A copy updates itself to the latest release</div>
</div>
<div class="card" style="left:324px; top:25px;">
  <div class="head"><div class="num">2</div><div class="title">Plan</div></div>
  <div class="file">planning.md, playbooks.md</div>
  <div class="line">Research how others solve it</div>
  <div class="line muted">Success criteria, thin slices</div>
</div>
<div class="card" style="left:624px; top:25px;">
  <div class="head"><div class="num">3</div><div class="title">Build</div></div>
  <div class="file">HARNESS.md, section 3</div>
  <div class="line">One slice at a time</div>
  <div class="line muted">A failing test first</div>
</div>
<div class="card" style="left:924px; top:25px;">
  <div class="head"><div class="num">4</div><div class="title">Check</div></div>
  <div class="file">code-check.md</div>
  <div class="line">Seven steps, one command</div>
  <div class="line muted">Nothing passes silently</div>
</div>

<div class="card" style="left:924px; top:392px;">
  <div class="head"><div class="num">5</div><div class="title">Test</div></div>
  <div class="file">testing.md</div>
  <div class="line">Only what a change reaches</div>
  <div class="line muted">FAILED, BLOCKED and FLAKY kept apart</div>
</div>
<div class="card" style="left:624px; top:392px;">
  <div class="head"><div class="num">6</div><div class="title">Review</div></div>
  <div class="file">code-review.md</div>
  <div class="line">Reads its own diff</div>
  <div class="line muted">No check loosened, trust boundaries read</div>
</div>
<div class="card" style="left:324px; top:392px;">
  <div class="head"><div class="num">7</div><div class="title">Ship</div></div>
  <div class="file">shipping.md</div>
  <div class="line">Main is production</div>
  <div class="line muted">Version, tag and a way back</div>
</div>
<div class="card" style="left:24px; top:392px;">
  <div class="head"><div class="num">8</div><div class="title">Learn</div></div>
  <div class="file">lessons.md</div>
  <div class="line">Lessons with evidence</div>
  <div class="line muted">Recalled when a task matches its cue</div>
</div>
</body>
</html>
```
