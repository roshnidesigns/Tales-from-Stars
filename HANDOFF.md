# Tales from the Stars - handoff

Pick-up notes for a new chat. The code is on GitHub and the README describes the
project itself, so this file holds only what those two do not carry: why things are
the way they are, what has already been tried, and what is still open.

**Read this first, then the README.**

## Where things are

| | |
|---|---|
| Folder | `~/Desktop/Claude Projects/night-sky` |
| GitHub | `roshnidesigns/Tales-from-Stars` (public) |
| Live | <https://roshnidesigns.github.io/Tales-from-Stars/> (GitHub Pages, from `main`) |
| Run locally | `python3 -m http.server 8000`, no build step |

Tags mark the milestones: `v1.0.0` (star chart), `constellation-ui-mapped` (visual
language), `added-visuals-graphics` (zodiac artwork), `added-audio-sync` (narration),
`final-version-working` (audio fix).

## What this is

An installation, not a website. A visitor stands in front of it and is walked through
seven scenes: the sky, the zodiacs, a spoken "yes", a spoken birth date, a card held
to a camera, the reveal of their sign, then topics to explore. Scene 0 is a genuine
star chart for a real place and time; the rest are the experience built on top.

Someone else is building the card-detection model. The camera stream is handed to
them on `window.__sky.cameraStream`.

## State, scene by scene

| | Scene | State |
|---|---|---|
| 0 | Sky | Done. Real astronomy, 2,072 stars, drag/pinch/zoom, click for stories |
| 1 | Zodiacs | Done. Slideshow, then "Ready to know your zodiac?", mic opens, yes or tap |
| 2 | Birthdate | Done. Asks aloud, mic opens itself, speech or typing, detects the sign |
| 3 | Card | Done. Names the shape number, speaks the line, opens the camera |
| 4 | Reveal | Done. Sign named, three characteristics pinned to stars |
| 5 | Gestures | **Not built.** Artwork in `assets/explore/`, narration mapped |
| 6 | Creature | **Not built.** Idea: the figure takes its animal shape and moves slowly |

## Decisions, and why

Things that look arbitrary but are not. Changing them without knowing why will
reintroduce a problem that was already solved.

**Each narration clip gets its own `Audio` element.** Reusing one caused both faults
the user reported: assigning a new `src` to an element that is still playing aborts
the previous load and fires an error for it asynchronously, so the new clip's handler
received the old clip's failure and skipped it (later scenes went silent), and
calling `play()` straight after setting `src` starts before enough data has arrived
(the opening word was clipped). Playback now waits for `canplay` and seeks to zero.

**Glyphs are on the zodiac only.** The five circumpolar figures keep lines, names and
stories but stay plain. With glyphs they crowded the north and competed with the
twelve the project is about.

**Glyph size tracks zoom only part way, and stars and glyphs have different floors.**
Pinning to angular size turns glyphs into stickers when zoomed in; pinning to screen
size crowds them when zoomed out. Stars are floored at their default and only grow,
because the wide view is where you want the whole sky.

**The shape number is derived, not tabulated.** The cue table's Shape # is the order
the Sun travels the signs, which `ZODIAC_ORDER` already holds, so the number is a
position in that list. There is deliberately no second table to keep in step.

**It opens on a dark sky, not the current hour.** Arriving at 3pm to a washed-out
blue sky is a poor introduction. If it is already dark, the real moment is used and
the clock keeps running.

**Everything degrades rather than dead-ends.** A stalled clip cannot hide the gate
(watchdog per clip), a blocked mic or camera says so and the scene still advances,
and the seven signs with no recording stay silent rather than borrowing another
sign's voice. This matters because the thing runs unattended.

**`data/constellations.json` carries Stellarium's GPL-2.0-or-later**, not the
project's MIT, because the line figures derive from their sky culture. Flagged rather
than quietly relabelled. The IAU groupings are fact, so the lines can be rebuilt by
hand if that is ever inconvenient.

## Gotchas that cost time

**GitHub Pages often misses the push webhook.** Twice a push did not trigger a build
and the live site silently stayed on the old commit. If a change is not live, check
the build before suspecting the code. It can be kicked from Settings > Pages or the
API.

**`python3 -m http.server` sends no `Cache-Control`.** Chrome caches `app.js`
heuristically, so a normal reload can serve the old file. This caused repeated "the
audio does not play" confusion when the fix was already in place. Hard reload with
shift-cmd-R.

**The Browser pane is a misleading test environment.** A hidden tab throttles timers
(a 5s interval measured as 6s) and blocks microphone and camera capture outright. The
user has asked not to open the browser anyway.

**Headless Chrome fast-forwards timers but not media playback**, so `ended` never
fires and anything waiting on audio (the scene 1 gate) never appears there. Several
headless runs also hung and had to be killed. `--dump-dom` against a state attribute
is the reliable trick; screenshots are fine for layout.

**`.git` is about 900 MB larger than it needs to be.** An early accidental commit of
the 909 MB source video was amended away, so the blob is unreachable and was never
pushed, but it still sits in the local object store. `git gc --prune=now` reclaims it
safely.

**`assets/backdrop.mp4` is 90 MB**, near GitHub's hard 100 MB file limit and
permanent in history. The user chose this over a lossless trim. `PresetPassthrough`
in `avconvert` trims without re-encoding if it ever needs to shrink.

**Some CSS blocks are duplicated inside `@media (max-width: 640px)`.** Anchor-based
edits matched text inside those queries. Harmless on desktop because every primary
rule is top-level, but the nested copies re-assert desktop values on narrow screens,
so mobile will look slightly off. Not yet cleaned up.

## Working preferences

- Use "-" rather than em or en dashes in app copy and READMEs.
- Do not open the browser pane. Headless screenshots are acceptable.
- Push when asked; leave the Pages build alone unless asked.
- The user iterates fast and in small messages. Prefer one change done and verified
  over a batch.
- Verify by instrumenting rather than asserting. Measured numbers have repeatedly
  caught things that looked right.

## Open threads

- **Scenes 5 and 6.** Artwork is in `assets/explore/` (career, friendship, love). The
  15 `topic_<sign>_<love|work|friendship>` clips are recorded and unwired.
- **A 15-second re-prompt** on the scene 1 gate if nobody answers. The user asked,
  then said to stop before it was built.
- **Only 5 of 12 signs have audio** for reveal and shape-pick (Aries, Taurus, Gemini,
  Cancer, Libra). The other seven are silent by design until recorded.
- **`seq_03b_birthdate_retry.mp3`** is recorded and unwired, for a birth date that is
  not understood.
- **Scene 5 and 6 narration mapping is a guess.** The `seq_04` / `seq_05` / `seq_06`
  numbering does not line up with the scene numbers; worth confirming.
- **The audio has never been checked by ear in this chat.** The mechanism behind the
  clipping and skipping is fixed and verified structurally, but nobody has listened.
- **Mobile CSS cleanup**, per the duplicated-blocks gotcha above.

## Update log

- **2026-10-18** - Rewrote the README for the full project. Created this handoff doc.
  Fixed narration clipping and skipping (fresh `Audio` element per clip, start on
  `canplay`). Moved the listening pill to the scene 1 gate and removed "Tap to speak"
  from scene 2. Moved the scene switcher to the top. Tagged `final-version-working`.
- **2026-09-10** - Built the whole thing from scratch in one long session: star
  chart, horizon view, glyph artwork, zodiac artwork, backdrop footage, the scene
  switcher and scenes 1 to 4, narration, microphone and camera.
