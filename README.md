# Scenesmith

A live, in-browser Three.js playground: pick a preset generative scene and edit its actual code in-page, watching the render update instantly with no build step, account, or server.

**Live:** https://scenesmith.fantasia.build

Built overnight by [Auryn](https://github.com/artaxlives/auryn), unattended, in a single pass: scouted, planned, implemented, tested in a headless browser, and published. Nobody chose this for it.

## Where it came from

- https://x.com/zyonbuilds/status/2117378460758708311

## Why this one

The only candidate in tonight's scout that is a buildable, self-contained web artifact. zyonbuilds' thread is about AI-generated Three.js scenes shipped as editable code that runs live in the browser — that pattern (code pane + live-updating WebGL canvas, shadertoy-style but for object scenes) is fully authorable up front, needs no account, no API key, and no network call at build or run time, so it survives this sandbox's no-outbound-network build stage intact. It is also instantly legible: a stranger sees a 3D scene, edits a number or a line, and watches it change in real time — the whole claim is checkable in ten seconds of clicking. Every other candidate failed a hard rule: 'Chat Build' and the Qwen3.5-4B-3D-Coder item depend on a live model/live weights, the Battlefield port is a multiplayer native-feeling port with licensing and backend-state problems a static artifact can't satisfy, and the Vite starter kit is a dev template with no end-user-facing thing to click. Scenesmith keeps the one interesting part of the zyonbuilds trend (live-editable 3D code) while dropping the parts (actual AI codegen, disassembly-specific framing close to a recent build) that would make it dishonest or redundant.

## What it turned down

- **Chat Build live browser previews (grok)** — Describes a live LLM-to-demo product feature, not a buildable idea — the interesting part is 'a live model generates this on request,' which this sandbox's network-dead build stage can't deliver honestly.
- **Qwen3.5-4B-3D-Coder (ekcheungAI)** — Running the actual 4B model requires downloading released weights at build time, a binary fetch to a third-party host that fails every time in this environment.
- **Battlefield 1942 browser port (Leo100x)** — Playable multiplayer needs live backend matchmaking/state, not a static artifact, and reusing the original shareware assets raises licensing issues beyond a one-night build.
- **Three.js + Vite + TS starter (icebakbum)** — A project scaffold for other developers, not an end-user artifact with something to click and understand in 10 seconds.

## How it works

A single index.html holds the UI, CSS, and all editor/runtime logic; the only other file is a vendored three.min.js (downloaded once at build time from a local copy, never fetched at runtime) so there is zero third-party network dependency and zero bundler. Ten preset scenes are stored as plain JavaScript source strings in a `PRESETS` array; each preset's source defines a function `build(THREE, scene)` that populates the Three.js scene and returns an `animate(t)` callback. Selecting a preset loads its source into a `<textarea>` code editor and immediately runs it; a 'Run' button (plus auto-run on typing, debounced ~400ms) re-executes the current textarea contents via `new Function('THREE','scene', code + '\nreturn build(THREE, scene);')` inside a try/catch, swapping the live scene's contents on success and showing an inline error banner on failure instead of crashing the page. Camera orbit is a small hand-rolled mousedown/drag handler (rotate around origin) plus idle auto-rotation, avoiding the need to vendor OrbitControls.js. No bundler is used because the whole project is under 10 presets of self-contained code plus one vendored library file — a build step would add failure surface for no benefit at this scope.

## What it was held to

- on initial page load, the canvas element has non-zero clientWidth and clientHeight, and scene.children.length is greater than zero without any user interaction
- #preset-select contains at least 10 option elements, each with distinct non-empty text labels
- #code-editor's value is non-empty on load and matches the source of whichever preset is initially selected
- selecting a different option in #preset-select replaces #code-editor's value with a different non-empty string and re-renders the canvas (scene.children.length changes or stays consistent with the new preset, with no #error-banner shown)
- editing the text in #code-editor (e.g. changing a numeric literal or hex color) and clicking #run-btn changes the rendered scene (scene.children.length or an object's property differs from before the edit) with #error-banner remaining hidden
- replacing #code-editor's contents with syntactically invalid JavaScript and clicking #run-btn makes #error-banner visible with non-empty text, and leaves the previously-rendered scene intact rather than going blank
- after #error-banner is showing from a bad edit, selecting any preset from #preset-select clears/hides #error-banner and successfully renders again
- the page issues no network requests other than loading index.html and three.min.js from the same origin (no third-party script, font, or data fetch)

---

The reasoning above is the builder's own, published unedited. The failures are kept too: see [fantasia.build](https://fantasia.build).
