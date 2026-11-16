# Eclipse Explorer — Build Prompt for Claude Code

> Paste this whole file into Claude Code (or keep it in the repo and say "follow PROMPT.md").
> Build it **step by step**: finish and test each step before starting the next. Each step gets its own branch and pull request on **codinglucas/eclipse-explorer** (see section 0).

---

## 0. GitHub setup (do this first)

All work lives on the **codinglucas** GitHub account (personal projects), **not** lucasravelino.

- **Repo:** `codinglucas/eclipse-explorer` (public). If it doesn't exist yet, create it with `gh repo create codinglucas/eclipse-explorer --public --description "3D learning simulator that teaches kids (6-8) how solar and lunar eclipses work"`.
- **Commit identity** (set locally in this repo; uses the GitHub no-reply address):
  ```bash
  git config user.name "codinglucas"
  git config user.email "113028485+codinglucas@users.noreply.github.com"
  ```
- **Never write a personal email into any file in this repo.** That includes commit messages and PR descriptions; use the no-reply address above wherever an email is needed.
- Before pushing, confirm `gh auth status` shows the **codinglucas** account. If it shows another account, stop and ask me to switch (`gh auth switch` / `gh auth login`).
- **First commit on `main`:** this `PROMPT.md` plus an empty `index.html` and a short `README.md` stub, with message `chore: project setup and build prompt`.
- **Workflow for every build step in section 11:**
  1. Create a branch from `main`: `step-N-short-name` (e.g. `step-1-scene-skeleton`).
  2. Build and test the step, committing as you go with clear messages (`feat:`, `fix:`, `docs:`, `chore:`).
  3. Push and open a **pull request** into `main` with `gh pr create`. The PR description says what the step adds, how to test it (open `index.html`, what to click), and a checklist of the step's ✅ checks.
  4. Show me the PR link and wait for my OK before merging (`gh pr merge --squash --delete-branch`), then pull `main` before starting the next step.
- Never commit secrets, `node_modules`, or OS junk; add a `.gitignore` (`.DS_Store`, `Thumbs.db`, `*.log`).
- Optional, final step: enable **GitHub Pages** from `main` / root so the sim is playable at `https://codinglucas.github.io/eclipse-explorer/`, and put that link at the top of the README.

---

## 1. What we're building

An interactive **3D learning simulator** that teaches kids aged **6–8** how **solar** and **lunar eclipses** work.
It is a guided simulator, not a scored game: two friendly guide characters walk the child through hands-on experiments, and the child *causes* the eclipses by moving the Moon.

- **Language:** English only
- **Platform:** desktop browsers and tablets (touch must work well)
- **Tech:** a **single `index.html` file** with vanilla JavaScript + **Three.js** (ES module from a CDN, e.g. `https://unpkg.com/three@0.16x/build/three.module.js` via an import map). No build step, no React, no npm. It must work by double-clicking the file.
- **Code layout inside the file:** clearly separated sections with comment headers — `CONFIG`, `SCENE`, `BODIES`, `LIGHT & SHADOWS`, `INTERACTION`, `GUIDES`, `LESSONS`, `UI`, `MAIN LOOP`. Keep all lesson text in one `LESSONS` data array so it's easy to edit without touching logic.

## 2. Core teaching idea

Both eclipses need the **Sun, Earth and Moon in a straight line**. The only difference is **who is in the middle**:

| | Solar eclipse | Lunar eclipse |
|---|---|---|
| Who's in the middle? | The **Moon** | The **Earth** |
| Whose shadow? | Moon's shadow falls on Earth | Earth's shadow falls on the Moon |
| Moon phase | New moon | Full moon |
| What you see | Sun gets covered, sky goes dark | Moon turns coppery red |
| Safe to look? | **NO** — only with eclipse glasses | Yes, totally safe |

Every lesson should come back to the question: **"Who's in the middle?"**

## 3. The guide characters

Two original characters (simple, cute, built from basic shapes/SVG — do **not** copy any existing cartoon character). They appear at different times and explain different things.

### ⭐ Stella the Star
- A small glowing yellow star with a face, gently pulsing/twinkling.
- **Explains light-related things:** the Sun is a star that makes its own light, light travels in straight lines, objects block light and make shadows, why the Moon "shines" (it reflects sunlight), why the eclipsed Moon turns red (sunlight bending through Earth's air, like a sunset).
- Appears on the **left** side of the screen, floats in with a sparkle.

### 👩‍🚀 Captain Kai the Astronaut
- A little astronaut in a round helmet with a visible smiling face, waving.
- **Explains the "being there" things:** the Earth and Moon, how the Moon orbits Earth, what people on the ground see during an eclipse, eclipse safety (eclipse glasses!), the "space is really big" scale reveal, and the final recap quiz.
- Appears on the **right** side of the screen, floats in like they're in zero gravity.

### Rules for both
- Only **one guide talks at a time**; the other is hidden or small and dimmed.
- Each guide speaks in a **speech bubble with one short sentence at a time** (max ~15 words, simple vocabulary for ages 6–8), then shows a big **"Next ➜"** button.
- **Read-aloud:** a 🔊 button on every bubble uses the browser's `speechSynthesis` to read the text. Give Stella and Kai slightly different voice pitch/rate so they sound different. Must fail gracefully if speech isn't available.

## 4. "Explore more!" on every explanation

Every speech bubble has an **"Explore more! 🔭"** button next to "Next".

- Tapping it **expands the bubble** with 1–3 extra sentences of deeper detail (still kid-friendly, but a little more advanced — good for curious kids or a parent reading along).
- It can be tapped once; then it changes to **"Show less"**.
- Optional extra: some "Explore more" entries also trigger a small visual in the 3D scene (e.g. highlight the shadow cone, show a light ray line, flash the Moon's orbit path).
- In the `LESSONS` data, every step has this shape:

```js
{
  id: "solar-blocking",
  guide: "kai",            // "stella" | "kai"
  text: "The Moon is blocking the Sun!",
  more: "The Moon is 400 times smaller than the Sun, but the Sun is about 400 times farther away. That's why they look the same size in the sky!",
  action: null,            // optional: what the child must do before Next appears, e.g. "alignSolar"
  sceneEffect: null        // optional: e.g. "showShadowCones", "groundView", "redMoon"
}
```

## 5. The 3D scene

- **Sun** on the left: bright glowing sphere (emissive material + soft glow sprite/halo). It is the light source — a `DirectionalLight` (or `PointLight`) at the Sun's position.
- **Earth** in the middle-right: cartoon Earth (blue sphere with simple green continent shapes — procedural canvas texture is fine, no remote images), slowly spinning.
- **Moon**: grey sphere with a few crater bumps (canvas texture), on a visible **dotted circular orbit** around Earth.
- **Background:** dark blue/purple space with a starfield (points).
- **Real shadows:** enable Three.js shadow maps. Earth and Moon both cast and receive shadows, so eclipses come from **actual shadows**, not fake overlays. Fit the shadow camera tightly around the Earth–Moon system so shadows stay sharp.
- **Shadow cones (teaching aid):** semi-transparent dark cones extending from Earth and Moon away from the Sun. Hidden by default, turned on by specific lessons.
- **Style:** bright, soft, friendly colors; chunky shapes; no tiny text in the scene. **Not to scale** by default (see Scale toggle).
- **Camera:** gentle orbit controls (limited zoom/rotate so kids can't get lost) + a **"Reset view"** button.

## 6. Interaction: moving the Moon

- The child **drags the Moon** (mouse or touch) along its orbit; the Moon stays locked to the orbit circle.
- Also provide two big arrow buttons **◀ ▶** to move the Moon, for kids who find dragging hard.
- **Snap assist:** when the Moon is within **~10°** of a perfect alignment:
  - Solar alignment = Moon between Sun and Earth.
  - Lunar alignment = Moon directly behind Earth (Earth between Sun and Moon).
  - Smoothly animate the Moon into exact alignment, play a soft "ding", show a gentle glow on the line Sun–Earth–Moon.
- When it's close but not yet snapped, the current guide can encourage: "Almost! Keep going!"

## 7. Eclipse effects

### Solar eclipse
- In space view: Moon's shadow visibly lands on Earth (a dark spot).
- Then a big button **"See it from Earth! 🌍"** switches to a **ground view**: a simple landscape (hills, a tree, a house silhouette) with the Sun in the sky. Animate the Moon's dark disc sliding over the Sun → sky darkens to dusk, stars appear, the Sun's corona glows around the edges. Then animate it back.
- Captain Kai **must** show the safety message here: **"Never look at the Sun without special eclipse glasses!"** with an eclipse-glasses icon. This message cannot be skipped by "Next" for at least 2 seconds.

### Lunar eclipse
- In space view: Moon slides into Earth's shadow and **slowly turns coppery red** (animate the Moon material color/emissive).
- Ground view option: night sky with a red Moon, Kai notes it's safe to look at.
- Stella explains the red color (sunlight bending through Earth's air, like sunsets). Her "Explore more" can go deeper.

## 8. Lesson flow (guided)

A top progress bar with chapter dots. Chapters:

1. **Meet the family** — Stella introduces the Sun ("I'm a star, just like the Sun!"). Kai introduces Earth and Moon. Child taps each body to see a one-line fact.
2. **Light and shadows** — Stella: light goes in straight lines; anything in the way makes a shadow. Turn on shadow cones.
3. **The Moon's trip** — Kai: the Moon travels around Earth about once a month. Child drags the Moon all the way around once (track progress). Optional: show the Moon's phase from Earth in a small corner window.
4. **Make a solar eclipse!** — Task: put the Moon in the middle. Snap → shadow on Earth → ground view → safety message.
5. **Make a lunar eclipse!** — Task: put the Earth in the middle. Snap → red Moon → Stella explains red.
6. **Who's in the middle? (recap quiz)** — Kai shows 3–4 picture questions with big tappable answers (e.g., two diagrams: "Which one is a solar eclipse?"; "Is it safe to look at a lunar eclipse?"). **No fail state:** wrong answers get a gentle hint and another try; right answers get confetti and a cheer.
7. **Free play unlocked** 🎉 — a "Free Play" mode where the child can move the Moon freely and trigger eclipses anytime. Guides only pop up briefly when an eclipse happens.

The child can **go back** to any completed chapter from the progress bar. Remember progress with `localStorage` (wrapped in try/catch; works fine without it).

## 9. Scale toggle

- A toggle button: **"See the real sizes! 🔭"**.
- When on, smoothly animate to **true relative sizes and distances** (Moon a tiny dot far from Earth; Sun enormous / very far away — compress the Sun distance as needed but show it's huge, e.g. a giant glowing edge at the screen side with a label "The Sun is WAY over here →").
- Captain Kai pops up: **"Space is really, really big!"** with an "Explore more" about real numbers (Moon ~384,000 km away; you could fit about 30 Earths in between).
- Toggle off animates back to the friendly, not-to-scale view. Snap assist and lessons should be disabled while real scale is on.

## 10. UI & accessibility

- **Big tap targets** (min 56 px), rounded buttons, high contrast, a friendly rounded font (e.g. "Baloo 2" or "Fredoka" from Google Fonts).
- **Minimal text** on screen at any time.
- Works in **landscape on tablets**; on narrow screens, speech bubbles go to the bottom.
- Buttons: Next, Explore more, 🔊 Read aloud, ◀ ▶ Moon arrows, Reset view, Real sizes toggle, Free play (after unlock), a small 🔇 sound-effects toggle.
- Simple sound effects via the Web Audio API (no audio files): ding on snap, whoosh on guide entrance, cheer on correct answer. Start audio only after a user tap.
- Respect `prefers-reduced-motion` (reduce floating/animations).

## 11. Build steps (in order; one branch + PR per step, see section 0)

1. **Scene skeleton** — `index.html` with Three.js via import map; Sun, Earth, Moon, orbit line, starfield, lighting, real shadows, camera controls, reset view. ✅ Check: it opens by double-click and shadows render.
2. **Moon interaction** — drag along orbit (mouse + touch), ◀ ▶ buttons, alignment detection, snap assist with ding + glow.
3. **Eclipse effects** — shadow cones toggle; solar shadow on Earth; red lunar Moon; ground view for both eclipses with sky darkening/corona.
4. **Guides** — Stella and Kai visuals, enter/exit animations, speech bubbles, Next, Explore more/Show less, read-aloud with different voices.
5. **Lessons** — `LESSONS` data array + lesson engine (steps, required actions, scene effects), progress bar, chapters 1–5.
6. **Recap quiz + Free play** — chapter 6 quiz with no fail state, confetti; chapter 7 free play unlock; localStorage progress.
7. **Scale toggle** — real-sizes animation and Kai's "Space is big" moment.
8. **Polish** — fonts, sounds, reduced motion, tablet layout, performance check (aim for smooth 60 fps on a mid-range tablet), and a final read-through of all lesson text for 6–8 year-old reading level.

After all steps, finish the `README.md` (in its own PR): what it is, the GitHub Pages link, how to open it locally, how to edit lesson text, and a screenshot placeholder.

## 12. Content accuracy checklist

- Solar eclipse → new moon; lunar eclipse → full moon. ✔
- Eclipses don't happen every month because the Moon's orbit is slightly tilted (mention only inside an "Explore more", not in the main flow). ✔
- Red lunar eclipse = sunlight refracted/filtered through Earth's atmosphere. ✔
- Never look directly at a solar eclipse without proper eclipse glasses (except the brief moment of totality, but **do not** teach that exception to 6–8 year olds — keep the rule simple). ✔
- Lunar eclipses are safe to watch with your eyes. ✔
- The view is "not to scale" unless the real-sizes toggle is on — say so in a small label. ✔
