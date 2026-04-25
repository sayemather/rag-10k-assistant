# SynthLens

Upload a song, pick a time range, and get a deep AI breakdown of the synthesis, texture, and musical DNA of any sound. Built for learning music production.

No database, no accounts. One HTML file + one API proxy.

---

## Setup (required before first use)

Your Anthropic API key lives in Vercel as an environment variable — you set it once and never touch it again.

**Step 1 — Get an API key**
Go to [console.anthropic.com](https://console.anthropic.com) → API Keys → Create new key. It starts with `sk-ant-...`.

**Step 2 — Add it to Vercel**
1. Go to your project on [vercel.com](https://vercel.com)
2. Settings → Environment Variables
3. Add: Name = `ANTHROPIC_API_KEY`, Value = your key
4. Click Save → then go to Deployments → redeploy (or push any change to GitHub to trigger it)

That's it. The key never appears in your code.

---

## What you get

**Identity chips**
Synthesis type · Synth model · Era · BPM · Key · Genre

**Sound properties**
Role · Attack · Sustain · Brightness · Movement — all 2-3 word values

**Tags** — timbral descriptors at a glance

**Why this synth** — 3 bullets, each a specific sonic clue

**Listen for** — 3 bullets on what to train your ears on

---

## Getting audio from YouTube

The most reliable way is **yt-dlp** — a free command line tool.

### Install on Mac (one time)

Open Terminal (`Cmd + Space` → Terminal → Enter):

```bash
# Install Homebrew if you don't have it
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install yt-dlp and ffmpeg
brew install yt-dlp ffmpeg
```

### Download any song as MP3

```bash
yt-dlp -x --audio-format mp3 -o ~/Desktop/"%(title)s.%(ext)s" "YOUR_YOUTUBE_URL"
```

The MP3 saves to your Desktop. Drag it into SynthLens.

### Download just a section

```bash
yt-dlp -x --audio-format mp3 --download-sections "*0:29-1:15" -o ~/Desktop/"%(title)s.%(ext)s" "YOUR_YOUTUBE_URL"
```

Replace `0:29-1:15` with the timestamps you want.

---

## File structure

```
synthlens/
├── index.html       # The entire frontend app
├── package.json     # Tells Vercel this is a Node project
├── api/
│   └── claude.js    # Serverless proxy — calls Anthropic API with your env key
└── README.md
```

---

## Deploy (free, ~5 min)

### GitHub
1. New repository → name it `synthlens` → Public
2. Upload `index.html`, `package.json`, `README.md` to the root
3. Create `api/claude.js`:
   - Add file → Create new file
   - Type `api/claude.js` in the filename box (the slash creates the folder)
   - Paste the contents → Commit

### Vercel
1. [vercel.com](https://vercel.com) → sign in with GitHub
2. Add New Project → import `synthlens` → Deploy
3. Go to Settings → Environment Variables → add `ANTHROPIC_API_KEY`
4. Redeploy once to pick up the key

**To update:** replace any file in GitHub. Vercel redeploys in ~30 seconds.

---

## How to use it

1. Download your audio via yt-dlp or use any MP3/WAV you have
2. Drag it into the drop zone
3. **Drag the orange handles** on the waveform to set your trim range — or type start/end times manually
4. Hit **Analyze This Section**
5. BPM and key appear immediately from the algorithms
6. Full breakdown loads in ~5 seconds

---

## Tips

- **15–30 second clips** work best
- **Find a moment where your sound is prominent** — a full mix makes synthesis harder to identify
- **Spectrogram is a learning tool** — spread harmonics = FM/wavetable, clean narrow bands = subtractive
- BPM detection needs a rhythmic pulse to be meaningful
- Key detection assumes tonal music — noisy clips may give inaccurate results

---

## How it works

1. File decoded via Web Audio API
2. Waveform drawn to canvas — drag handles update trim start/end in real time
3. `OfflineAudioContext` slices the selected range
4. Cooley-Tukey FFT draws the spectrogram
5. Onset autocorrelation detects BPM
6. Krumhansl-Schmuckler chromagram detects key
7. Acoustic measurements sent to Claude via `/api/claude` serverless proxy
8. Claude returns structured JSON — rendered as chips, bullets, and tags

The proxy exists because browsers can't call the Anthropic API directly (CORS). It reads your API key from the Vercel environment variable so it never touches your code.

---

## Limitations

- **No stem separation** — analyzes the full mix. Find a moment where your target sound is isolated
- **Synth ID is a best guess** — treat specific model guesses as directional, not definitive
- **BPM needs a pulse** — ambient clips return a number but it may not be meaningful
- **Key assumes tonal music** — noisy or percussive clips may return an inaccurate key

---

## Stack

- Vanilla HTML/CSS/JS — no framework, no build step
- Anthropic API — Claude Sonnet
- Vercel — hosting + serverless proxy
- Web Audio API — decoding and slicing
- Cooley-Tukey FFT — frequency analysis + spectrogram
- Krumhansl-Schmuckler — key detection
- Onset autocorrelation — BPM detection
- Syne + Space Mono — fonts

---

## Cost

~$0.01–0.03 per analysis (text only, no audio attachment).

The API key lives in Vercel's environment — never in your code or git history.

---

## Local use

Open `index.html` directly in Chrome or Firefox. File loading, waveform trimmer, spectrogram, BPM, and key all work offline. The Claude analysis won't work locally (the proxy needs Vercel), but everything else does.
