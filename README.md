# apple-music-pipeline

Download a song from a link and have it appear in **Apple Music** automatically —
no drag-and-drop, no manual import.

```bash
addsong "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
```

That one command downloads the audio, tags it (title, artist, cover art), and drops
it into Apple Music's watch folder. Apple Music imports it on its own within a second
or two.

---

## What it does

```
addsong <youtube-url>
        │
        ├─ 1. yt-dlp downloads best audio  ──► temp .m4a file
        │
        ├─ 2. ffmpeg converts to AAC/m4a + embeds artwork & metadata
        │
        └─ 3. file is moved into Apple Music's "Automatically Add to Music" folder
                                   │
                                   └─► Apple Music auto-imports it. Done.
```

No background server. No AppleScript. Just a small wrapper script around two
well-known tools.

## Status

Planning / docs phase. See [ROADMAP](#roadmap) below and
[`ARCHITECTURE.md`](./ARCHITECTURE.md) for the full design.

## Requirements

- **macOS** with the **Music** app (this is a Mac-only project by design — see
  the Windows note in `ARCHITECTURE.md`)
- [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) — the downloader
- [`ffmpeg`](https://ffmpeg.org/) — audio conversion + tagging

```bash
brew install yt-dlp ffmpeg
```

## Install

Full first-time setup (including the one-time watch-folder verification) is in
[`SETUP.md`](./SETUP.md). The short version:

1. `brew install yt-dlp ffmpeg`
2. Put the `addsong` script on your `PATH` (e.g. `~/bin` or `/usr/local/bin`)
3. `chmod +x addsong`
4. Confirm Apple Music's watch folder exists (see SETUP).

## Usage

```bash
# single track
addsong "https://www.youtube.com/watch?v=..."

# (planned) a whole playlist
addsong --playlist "https://www.youtube.com/playlist?list=..."
```

## Defaults

| Setting        | Value                                                        |
|----------------|-------------------------------------------------------------|
| Audio format   | `m4a` (AAC) — Apple-native                                   |
| Artwork        | embedded from the video thumbnail                            |
| Metadata       | parsed from video title/uploader, cleaned up                 |
| Source         | YouTube (other yt-dlp sites likely work but aren't a target) |
| Import method  | Apple Music "Automatically Add to Music" watch folder        |

## How the "instant import" works

Apple Music watches this folder and imports anything dropped into it:

```
~/Music/Music/Media.localized/Automatically Add to Music.localized/
```

The pipeline's only job for the import step is to land a properly tagged file there.
Apple does the rest. (Path can vary slightly by macOS version / library name —
`SETUP.md` shows how to confirm yours.)

## Roadmap

- [ ] **Phase 1 — basics:** download a single YouTube URL → m4a → watch folder → confirm auto-import
- [ ] **Phase 2 — metadata:** clean ugly titles (`(Official Video) [4K]` etc.), reliable artist/title split
- [ ] **Phase 3 — convenience:** playlist support, duplicate awareness, better errors
- [ ] **Phase 4 — polish:** config file for output paths/format, logging

## Legal note

This tool downloads audio from third-party sites. Downloading copyrighted material
may violate those sites' Terms of Service and/or copyright law. Use it for content
you own, public-domain, or Creative-Commons material — and use your own judgment
for anything else. You are responsible for how you use it.
