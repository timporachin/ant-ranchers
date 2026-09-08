# Ant Ranchers

A two-minute animated short about how ants farm aphids — carrying them off a rose stem,
keeping them underground, stroking their abdomens with their antennae to draw honeydew,
and tearing the legs off ladybirds that come for the herd. Every behaviour in it is real;
the page under the player says which bits are which.

- **Watch it:** open `ants/index.html` (on GitHub Pages, `/ants/`). Play, pause, scrub, jump by
  chapter, or switch on **Narrate** for the spoken commentary.
- **The video file:** `ants/ant-aphid-farming.mp4` — 2:18, 1280×720, 30 fps, H.264 with an
  AAC narration track, about 13 MB. It has sound; the captions are burned in as well, so it
  still reads with the volume off.

## How it is built

There is no video source and no image assets. The film is drawn into a canvas by
`AntFilm.drawFrame(ctx, t)`, a **pure function of time** — no wall clock, and every bit
of texture and jitter comes from a seeded PRNG. That is what lets the web player and the
offline renderer produce byte-identical frames for the same timestamp.

| File | What it is |
| --- | --- |
| `ants/art.js` | Drawing primitives: ants, aphids, ladybirds, plants, soil, callouts, effects |
| `ants/scene.js` | The film — eight scenes, the camera, the caption track, `drawFrame` |
| `ants/player.js` | The player: clock, transport, chapters, narration, `?render=1` mode |
| `ants/index.html`, `ants/ants.css` | The page |
| `ants/narration.mp3` | The spoken commentary, one clip per caption on a single bed |
| `tools/render-video.mjs` | Walks the timeline in headless Chromium and encodes the MP4 |
| `tools/build-narration.mjs` | Speaks the caption track with Piper and lays out the audio |

## Re-rendering the video

```sh
node tools/build-narration.mjs                           # ants/narration.mp3
node tools/render-video.mjs --audio ants/narration.mp3   # the whole film, with sound
node tools/render-video.mjs --start 57 --end 84 --jpeg   # one scene, quick preview
node tools/render-video.mjs --scale 2 --out ants/1440p.mp4
```

Needs Playwright with Chromium, and an ffmpeg built with libx264. If there is no ffmpeg
on `PATH` the script fetches one with `pip install imageio-ffmpeg`. Playwright's own
bundled ffmpeg is a stripped VP8-only build and deliberately is not used.

The narration needs Piper (`pip install piper-tts`); the voice model downloads itself on
first run. Piper varies phoneme durations between runs by default, which would let the
voice drift out from under the captions, so synthesis is pinned with `--noise-w-scale 0`
and is reproducible.

Changing anything in `ants/art.js` or `ants/scene.js` changes the film, so re-run the
renderer to bring `ants/ant-aphid-farming.mp4` back in step with the page. Changing a
caption changes what is spoken, so rebuild the narration too.
