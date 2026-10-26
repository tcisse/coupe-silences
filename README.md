# Coupe Silences

Browser tool that removes dead air (silences) from a video before editing, built for the internal video team (CapCut workflow).

Everything runs client-side: the video never leaves the user's machine.

## How it works

1. **Analysis**: the audio track is decoded with WebCodecs ([Mediabunny](https://mediabunny.dev)) and reduced to one peak level per 10 ms window. This runs once per file.
2. **Plan**: silences are runs of windows below the threshold that last at least the minimum duration. Each silence is shrunk by a safety margin; everything else is kept. Recomputed instantly when sliders move, so the timeline updates live.
3. **Preview**: the source video plays in the page and jumps over cut regions.
4. **Export**: kept segments are re-encoded (H.264 + AAC when available) into an MP4. Video frames and audio samples go through the same time map, so audio and video cannot drift, whatever the number of cuts.

## Settings

| Setting | Default | Effect |
|---|---|---|
| Silence threshold | -35 dB | Higher cuts more (breaths, room noise). |
| Minimum silence | 0.6 s | Shorter pauses are kept. |
| Margin kept | 0.12 s | Audio kept on each side of a cut so word edges are not clipped. |

Presets: Doux (-40 dB, 1.0 s, 0.20 s), Normal (defaults), Nerveux (-32 dB, 0.3 s, 0.06 s).

## Browser support

- Chrome, Edge, Safari (recent): H.264 + AAC output, hardware encoding.
- Firefox: works, AAC through the WASM fallback encoder.
- If H.264 is not available the app falls back to VP9/HEVC/AV1 and warns that CapCut may not open the file.

## Limits

- The output MP4 is built in memory: fine for 10 to 20 min of 1080p, very long videos may exhaust the tab.
- HDR footage (iPhone) is exported as SDR.

## Development

Single static file (`index.html`), no build step. Dependencies load from jsDelivr, pinned:

- `mediabunny@1.62.1`
- `@mediabunny/aac-encoder@1.62.1` (lazy, only when the browser cannot encode AAC)

Run locally with any static server, for example `python3 -m http.server`, then open `http://localhost:8000`.

## Deployment

Pushed to `main` (through a PR), `.github/workflows/pages.yml` deploys the site to GitHub Pages.
