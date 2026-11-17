# Eclipse Explorer 🌑🌕

**▶ Play it now: https://codinglucas.github.io/eclipse-explorer/**

A 3D learning simulator that teaches kids aged **6–8** how **solar** and **lunar eclipses** work.
Two friendly guides, **Stella the Star** and **Captain Kai the astronaut**, walk the child through hands-on experiments. The child *makes* each eclipse by moving the Moon.

> Screenshot coming soon. Add one as `docs/screenshot.png` and show it here with `![Eclipse Explorer](docs/screenshot.png)`.

---

## The big idea: "Who's in the middle?"

Both eclipses need the Sun, Earth and Moon in a straight line. The only difference is **who is in the middle**:

| | Solar eclipse | Lunar eclipse |
|---|---|---|
| Who's in the middle? | The **Moon** | **Earth** |
| Whose shadow? | The Moon's shadow falls on Earth | Earth's shadow falls on the Moon |
| Moon phase | New moon | Full moon |
| What you see | The Sun gets covered, the sky goes dark | The Moon turns coppery red |
| Safe to look? | **No**: only with eclipse glasses | Yes, totally safe |

## What's inside

- **A real 3D scene** (Three.js). The eclipses come from **real shadows**, not painted-on overlays. A dotted orbit, a starfield, and optional shadow cones and light rays as teaching aids.
- **The Moon travels by itself.** It starts out orbiting slowly, with a big **Play / Pause** button between the ◀ ▶ arrows. A playing Moon glides straight through the line-ups; it never snaps and never completes a lesson task by itself.
- **Move the Moon** by dragging it (mouse or touch), with the big ◀ ▶ buttons, or with the arrow keys. Moving it pauses it, so it stays where you put it. Close to a perfect line-up, a paused Moon **snaps** into place with a soft chime. When a lesson says "your turn", the Moon pauses and waits.
- **An optional tilted Moon path, so eclipses are not every lap.** By default the path is flat and there is an eclipse on every lap. With ⚙️ **Tilted path** switched on, the Moon's path is tilted, and the line where it crosses the Sun–Earth plane slowly turns (once in about 11.74 laps, like the real eclipse year). While the Moon plays by itself, it usually passes above or below the line-up and nothing happens; about every 6th lap the crossing line points at the Sun and you get an eclipse season. When the Moon is paused or moved by hand, the path turns to the nearest eclipse season, so lining the Moon up always makes an eclipse and every lesson still works. The real tilt is about 5°; it is drawn at 45° because Earth and the Moon are drawn far too big for their distance.
- **Look from anywhere.** Drag empty space to turn the view all the way around, and from straight above to straight below. The **＋** and **−** buttons (or the mouse wheel, or a two-finger pinch) zoom in and out. **🎯 Reset view** swings back home.
- **🌍 View from Earth frame.** A small live picture of the sky that always matches the space view: daytime with the Sun while the Moon is on the Sun's side (the Moon only shows where it covers the Sun), night with the Moon and its phase while it is on the far side, and both kinds of eclipse as they happen.
- **View from Earth, full screen.** Tap the small 🌍 frame to open it big, in the **countryside** or the **city**. With no eclipse lined up it shows the same live sky as the frame. With the Moon snapped into a line-up it plays that eclipse:
  - **Solar:** the Moon slides over the Sun, the sky turns to dusk, and stars and the corona appear.
  - **Lunar:** a night sky where Earth's shadow turns the Moon red.
- **Safety first.** Before every solar eclipse view, Captain Kai explains why you must **never look at the Sun without eclipse glasses**. The message can't be skipped for 2 seconds.
- **Guides with speech bubbles.** One short sentence at a time, **Next ➜**, **Explore more! 🔭** for curious kids, **🔊 read-aloud** (a different voice for each guide), and a ✕ that closes the tutorial for good: no bubble comes back, also after a reload, until **Learning mode** is switched on again in ⚙️ (only the Sun safety talk still shows).
- **7 chapters**, played in order; the current one is named at the top:
  1. ☀️ Meet the family
  2. 🔦 Light and shadows
  3. 🌙 The Moon's trip
  4. 🌑 Make a solar eclipse!
  5. 🌕 Make a lunar eclipse!
  6. ❓ Who's in the middle? A picture quiz with no fail state: wrong answers get a gentle hint, right answers get confetti.
  7. 🎉 Free play
- **⚙️ Settings** (top right), with big switches and buttons:
  - **🎓 Learning mode:** turn it off to explore freely, with no lessons and no chapter title; the guides only pop up briefly when an eclipse happens. Turn it back on to continue the lesson where you left it.
  - **🌍 View from Earth:** show or hide the small sky frame.
  - **🌙 Moon speed:** 🐢 Slow, 🚲 Medium or 🚀 Fast. Picking one starts the Moon.
  - **🪐 Tilted path:** off by default (a flat path, an eclipse on every lap). On: the tilted path described above.
  - **🔦 Shadows:** show or hide the shadow cones (a dark inner shadow and a faint outer shadow for Earth and for the Moon).
  - **🌅 Sunset light:** shows sunlight skimming through Earth's air: the blue is bounced away and the red bends into Earth's shadow, which is why the eclipsed Moon turns red.
  - **🔊 Sound effects:** on or off.

  All of these choices are remembered.
- **Progress is saved** in the browser, and ↺ starts over (with a grown-up check).
- **Kid-friendly UI.** Big tap targets, the rounded Fredoka font, works on tablets (landscape and portrait) and phones, and respects **reduced motion**.

## Open it locally

It's a **single `index.html` file**, with no build step and no install.

1. Download or clone this repo.
2. **Double-click `index.html`.** It opens in your browser.

Notes:
- The first time, it needs an internet connection to load Three.js and the font from a CDN.
- Some browsers don't save progress for pages opened straight from a file (`file://`). Everything else works; the child just starts at chapter 1. To test saving locally, serve the folder with any static server, for example:

  ```bash
  python3 -m http.server 8000
  ```

  Then open http://localhost:8000.

## Edit the lesson text

All the words the guides say live in **one place**: the `LESSONS` section near the middle of `index.html`. You can change the text without touching any logic.

### `LESSONS`: the chapters

`LESSONS` is a list of chapters. Each chapter has `id`, `icon`, `title` and a list of `steps`. A step looks like this:

```js
{
  id: "solar-task",            // unique name for the step
  guide: "kai",                // "stella" (light things) or "kai" (being-there things)
  text: "Who's in the middle? Put the Moon between the Sun and Earth!",  // one short sentence, max ~15 words
  more: "Drag the Moon toward the Sun, or use the arrow buttons.",       // the "Explore more! 🔭" text
  action: "alignSolar",        // optional: what the child must do before Next appears
  sceneEffect: null,           // optional: e.g. "showShadowCones"
}
```

- **Actions:** `tapBodies`, `orbitOnce`, `alignSolar`, `alignLunar`, `redMoon`, `groundSolar`, `groundLunar`. Add `optional: true` to show Next right away.
- **Scene effects:** `showShadowCones`, `hideShadowCones`, `showLightRays`, `hideLightRays`, `moonStart`, `resetView`. One name, or a list of names.
- **Quiz steps** (chapter 6) add a `quiz` object with `answers` (each with `picture` or `emoji`, `label` and `correct`), plus a `hint` and a `praise`.

### Other text

- **`SCRIPTS`:** short lines for free play, the ground view (including the solar **safety talk**) and the real-sizes moment.
- **`BODY_FACTS`:** the one-line facts shown when the child taps the Sun, Earth or Moon in chapter 1.
- **`TASK_TEXT`:** small labels such as "Almost! Keep going!" and "You did it! 🎉".

Writing tips for ages 6–8:
- Keep each `text` to one short sentence (15 words at most).
- Name things ("the Moon", "Earth's shadow") instead of saying "it".
- Save extra detail for `more`.

## Code layout

Everything is in `index.html`, split into clearly marked sections:

`CONFIG` · `SCENE` · `BODIES` · `LIGHT & SHADOWS` · `INTERACTION` · `GUIDES` · `LESSONS` · `UI` · `MAIN LOOP`

- **Tech:** vanilla JavaScript plus [Three.js](https://threejs.org/) 0.160, loaded as an ES module through an import map. No frameworks, no npm.
- **Textures, sounds and characters** are all drawn or generated in code: canvas textures, Web Audio sound effects, and SVG guides. There are no image or audio files.
- **Developer helper:** `window.__eclipse` in the browser console gives you shortcuts for testing, for example `__eclipse.startChapter(3)`, `__eclipse.setMoonAngle(Math.PI)` and `__eclipse.resetProgress()`.

## Science notes

The simulator keeps the science right while staying simple for young kids:

- A solar eclipse happens at **new moon**; a lunar eclipse at **full moon**.
- Eclipses don't happen every month because the Moon's orbit is slightly **tilted**. This is mentioned only in "Explore more".
- The eclipsed Moon looks **red** because Earth's air bends and filters sunlight, like the light at sunset.
- **Never look at the Sun without proper eclipse glasses.** Regular sunglasses are not enough. The app keeps this rule simple on purpose.
- **Lunar eclipses are safe** to watch with your eyes.
- The view is **not to scale**: sizes and distances are squeezed so the eclipses are easy to see.

## How it was built

Built step by step from [PROMPT.md](PROMPT.md), with one pull request per step: scene, Moon interaction, eclipse effects, guides, lessons, quiz and free play, real sizes, and polish.
