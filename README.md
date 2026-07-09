# 🎬 video-social-edit

> A [Claude Code](https://claude.com/claude-code) skill that turns a raw screen recording into a polished, cropped, captioned social‑media clip — shot list to final render, no timeline scrubbing required.

<p align="center">
  <code>raw .mov</code> → <b>shot list</b> → <b>rough cut</b> → <b>reframe</b> → <b>text overlays</b> → <b>voice‑over + facecam + captions</b> → <code>ready to post</code>
</p>

---

You drop in a screen recording of a demo. Claude probes it, proposes a shot list, cuts the
dead air, reframes it to whatever aspect ratio the platform wants, burns in clean text
overlays, and — if you like — muxes in a voice‑over you record on your phone with a circular
face bubble and captions. Every stage is a real, re‑runnable `ffmpeg`/Pillow script you can
inspect and tweak; nothing is a black box.

Built from the hard‑won details of shipping real product demos — the ffmpeg gotchas that
usually cost you an afternoon are already encoded.

## ✨ What it does

- **Cuts the boring parts** — a shot‑list phase where you decide what stays, what speeds up (1.5× walking pace, 8× montages), and what gets dropped.
- **Reframes to any social format** — 9:16 (Stories / Reels / Shorts / TikTok), 1:1 (feed), 4:5 (portrait), 16:9 (YouTube / X). Hard crops, blurred backdrops, zooms, and smooth animated pans between panels.
- **Burns in text overlays** — dark/light pill, bottom bar, or brand‑accent styles, rendered as crisp PNGs (works even on ffmpeg builds without `drawtext`).
- **Adds voice‑over + facecam** — you record face + voice in one QuickTime take against the finished cut; the skill overlays a circular face bubble and muxes your narration.
- **Optional captions** — one caption per voice‑over line, timed to each segment.
- **Writes the voice‑over script for you** — one punchy line per segment, timed to its duration.
- **Watches your privacy** — flags frames with personal info (emails, autofill popups, personal tabs) so they never make the cut.

## 🚀 Quick start

Say something like:

> *"Edit `demo.mov` into a 9:16 Instagram Story — focus on the browser on the left, cut the dead moments, add text overlays."*

Claude walks the pipeline with you, pausing at each phase so you can steer:

1. **Probe & interview** — reads the source, asks target format + what's on screen.
2. **Shot list** — a reviewable table of cuts, speeds, focus, overlay text, and voice‑over.
3. **Rough cut** — a fast landscape preview so you can fix pacing *before* the expensive reframe.
4. **Reframe** — crops/pans to your target aspect ratio.
5. **Text overlays** — final render with burned‑in titles.
6. **Voice‑over, facecam & captions** *(optional)* — record narration, get a face bubble + captions.

Other things you can ask:

> *"Make a square LinkedIn clip from this, speed up the slow parts, title at the top."*
> *"Write me a shot list and voice‑over script for a 20‑second TikTok first — I'll approve before you render."*
> *"Add my voice‑over: here's the recording, put my face bubble top‑right with captions."*

## 📦 Installation

This is a Claude Code skill. Drop it into your skills directory:

```bash
git clone https://github.com/joanseg/video-social-editor.git \
  ~/.claude/skills/video-social-edit
```

Then just ask Claude to edit a video — the skill triggers automatically. (Restart Claude Code
if it was already running.)

### Requirements

| Tool | Why | Install |
|------|-----|---------|
| **ffmpeg** | all cutting, cropping, overlay, muxing | `brew install ffmpeg` |
| **uv** | runs the Pillow scripts in an isolated env | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **Pillow** | renders title/caption PNGs (pulled in on demand by `uv run --with pillow`) | — |

No build step, no Node, no cloud. Everything runs locally.

## 🎯 Supported formats

| Format | Dimensions | Best for |
|--------|-----------|----------|
| `9:16` | 1080×1920 | Instagram Stories/Reels, TikTok, YouTube Shorts |
| `1:1`  | 1080×1080 | Instagram & LinkedIn feed, X |
| `4:5`  | 1080×1350 | Instagram portrait feed |
| `16:9` | 1920×1080 | YouTube, LinkedIn landscape, X |

## 🧩 How it's built

```
video-social-edit/
├── SKILL.md                      # the workflow Claude follows
├── references/
│   ├── social-formats.md         # format specs + per-platform durations
│   ├── crop-geometry.md          # hard crop, blurred backdrop, zoom, animated pan math
│   ├── overlay-styles.md         # title style presets + the 2-line rule
│   └── voiceover-facecam.md      # QuickTime recording steps, bubble/caption layout
├── scripts/
│   ├── generate_titles.py        # title-card PNGs (4 styles, adaptive sizing)
│   ├── generate_captions.py      # bottom caption PNGs from VO lines
│   ├── make_circle_mask.py       # circular alpha mask + ring for the facecam bubble
│   ├── add_voiceover.sh          # mux face+voice recording → bubble + audio
│   └── add_captions.sh           # burn captions timed to segments
└── evals/
    └── evals.json                # test prompts used to benchmark the skill
```

Progressive disclosure: `SKILL.md` holds the workflow; the reference files load only when a
phase needs them; the scripts run without ever entering the context window.

### A few of the encoded gotchas

- Uses `scale=…:-2` (not `-1`) so H.264 always gets even dimensions.
- Caps every concat with `-t` to trim the frozen tail that keyframe rounding leaves behind.
- Renders text with **Pillow → PNG → overlay** because many ffmpeg builds ship without `drawtext`/libass.
- `format=yuv420p` on the **last** overlay only, for universal playback.
- Animated pans use a time‑expressed `crop` x‑offset with a snappy 0.667 s sweep.

## 🎙️ Voice‑over & facecam

You record **face + voice together in one QuickTime "New Movie Recording"** while the finished
video plays — so audio and face are already in sync. The skill then:

- crops your webcam to a circle, adds a ring, and drops it in the corner you pick;
- uses the recording's audio as the voice‑over track;
- optionally burns captions along the bottom.

> **Layout tip the skill enforces:** with captions on, put the face bubble in a *top* corner —
> face up top, captions along the bottom, the standard reel layout.

## 🤝 Contributing

Issues and PRs welcome — new format presets, overlay styles, or platform‑specific tweaks are
all fair game. The skill is designed to be read and extended: each script is small, single‑purpose,
and does one stage of the pipeline.

## 📄 License

MIT — see [LICENSE](LICENSE).
