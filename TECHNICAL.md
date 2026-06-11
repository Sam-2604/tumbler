# TUMBLER - Technical & Design Notes

This document explains how the game is built and, more importantly, why each decision was made. The entire game is one HTML file (`tumbler.html`, ~1400 lines); everything below lives in it.

## 1. One file, no build

The game is a single self-contained HTML document: CSS, markup, and JS in one file, with only Google Fonts as an external dependency.

**Why:** the game is small enough that a build system would add cost without benefit. One file means trivial deployment (copy it anywhere), trivial archiving, and the whole program is readable top to bottom. The discipline this forces - clear section banners, small systems - turned out to be a feature, not a constraint.

## 2. The 9:16 stage

`#app` is aspect-locked to 9:16: `width: min(100vw, 56.25dvh)`. On phones it fills the screen; on desktops it letterboxes into a centered portrait card. All type is sized with container-query units (`cqw`), so text scales with the stage, not the window.

**Why:** the backdrop art is 9:16 portrait. Rather than crop art per aspect ratio (endless edge cases), the game owns one canvas shape everywhere. Every layout bug becomes testable at a single aspect ratio, and phone/desktop differ only in absolute size.

## 3. Two canvases plus DOM

- `#scene` - the backdrop (image, weather effects, legibility gradient)
- `#lock` - the interactive mechanism, redrawn every frame
- DOM overlays - HUD, dialogue, story pages, settings, buttons

**Why:** the lock needs a per-frame repaint; the scene mostly doesn't. Splitting them isolates the hot path. Text lives in the DOM, not canvas, so it reflows, scales, scrolls, and stays selectable by accessibility tools - canvas text would have to reimplement all of that.

The art loads lazily per chapter. If an image is missing or slow, each scene has a procedural canvas painter as fallback, so the game never shows a blank screen (and the repository ships without the art at all).

## 4. Data-driven chapters

All ten chapters live in one `LEVELS` array. A chapter is data: palette, ambience spec, music spec, mechanism phases, tutorial line, dialogue beats, intro pages, outro pages. The engine code contains no per-chapter logic.

**Why:** the game is mostly content. Keeping content as data made the two full story rewrites cheap (text-only diffs), and adding a chapter is a checklist, not a programming task.

### The lock engine

Three mechanics share one phase system (`mech.phases[]` - chapter 10 chains all three):

- **pins** - rise/fall rates, tolerance band, plus modifiers: `sequence` (ordered), `tension` (overpush binds), `decoy` (false bands that click then slip), `hideBand` (pick blind by haptic/shimmer), `shimHP` (limited tool breaks), `interfere` (the rival knocks set pins loose)
- **gates** - self-spinning warded rings, `surge` ties speed to a tide swell
- **dial** - sweeping needle, arcs narrow per chapter, `drift` moves targets with the water

`strict` (chapter 9 dial, chapter 10 wards and master ring): one miss releases everything set in that layer.

**Why strict late:** early chapters teach; late chapters should bite. Resetting the layer (not the level) punishes greed without erasing the run. And in chapter 10 the difficulty is also fiction: the `interfere` modifier *is* Rooke fighting through the brass, and the final unopposed dial *is* him letting go - the player feels the surrender before the text confirms it.

## 5. Dialogue as a story moment

When a character speaks mid-lock, the game pauses: the clock holds, the lock dims and blurs, and a tap anywhere continues. Dismissal ignores taps for 500ms after the box appears.

**Why:** a non-blocking auto-fading version was built first and rejected - the box covered a live mechanism and reading competed with playing. Pausing costs nothing (the timer holds) and makes the words an event. The 500ms guard exists because players in the middle of tapping a gate would dismiss the line before seeing it.

**Beat ordering rule:** lines that answer each other must fire on triggers with a guaranteed order (pin counts: `set 1 → 2 → 3`), never on a wall-clock timer racing a player-speed trigger. A fast player once saw the retort before the line it answered.

## 6. The story pager

Chapter text renders in a scrollable overlay, but the design rule is **no page may need scrolling on a phone-sized stage** - long content is split into more pages instead. A bobbing chevron appears on the rare small screen where a page still overflows, and disappears at the bottom.

Navigation is always available: back through pages, intro back to the title, prev/next chapter from any intro, a persistent back button during play that re-opens the chapter text and resumes the same lock state, and restart-chapter returns to the chapter text rather than dumping the player into the mechanism.

A plain-words retelling of the whole plot ("The Story") unlocks on the title screen after chapter 10 - the in-fiction prose is deliberately spare, so the game offers its own explainer once the player has earned the ending.

## 7. Audio: everything synthesized

There are no audio files. WebAudio generates all of it:

- **Buses:** separate music and SFX gain nodes (independent volume sliders); the SFX bus runs through a 4.5kHz lowpass so clicks never bite
- **Ambience:** per-chapter filtered-noise beds and detuned drones (rain is just bandpassed noise with a slow LFO)
- **Music:** a small generative engine - scale + chord progression + bpm per chapter, a per-chapter **motif** (a degree pattern transposed onto the current chord), one of three lead voices (*bell*, *pluck*, *air*), and an arpeggiated pulse mode for tense chapters; intensity rises when the clock runs low

**Why generative:** music files would multiply the download size and break the single-file character of the project. The motif system exists because v1 (random plucks over pads) made every chapter sound like the same tune - a recurring melody per chapter is what makes them distinct, for roughly thirty lines of code.

## 8. Persistence and state

Three guarded localStorage keys: progress (`tumbler_progress_v2`), settings (`tumbler_settings_v1`), tutorial-seen (`tumbler_howto_v1`). Global state is a single `state` string (`title / story / play / result`) plus the current `lock` object; pause is computed in one place (`syncPause()`) from "dialogue open OR settings open".

## 9. Assets

Backdrops are 1536x2752 JPEG (quality 85), converted from PNG originals - 94MB to 12MB with no visible loss for painted art, which is the difference between unpublishable and instant-loading. The art is deployed with the game but excluded from the repository; the canvas fallbacks keep a bare clone runnable.

## 10. Testing

- Syntax gate: the script block is extracted and parsed with JavaScriptCore on every change
- Playwright (headless Chromium) suites: text overflow/overlap sweeps across six viewports, story-page no-scroll audits, win/fail/mute/result flows, navigation, strict-reset behavior, and a **full automated 10-chapter walkthrough** (win chain, outro pages, save progression, art loading)
- Writing rules enforced by grep: no em/en dashes, vocabulary bans from the plain-language pass

**Why this shape:** with no framework and no unit-test seams, the honest test is the browser. Driving the real game loop end to end caught the bugs that mattered: overlay taps swallowing input, HUD overlap regressions, dialogue order races.

---
<em>Samarth Goradia</em>
