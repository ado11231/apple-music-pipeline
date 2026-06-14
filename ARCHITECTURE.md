# Architecture & Design

This document explains *how* `apple-music-pipeline` works, *why* it's built this way,
and the edge cases that matter. If you just want to use it, read the README.

## Goal

Turn a YouTube link into a tagged track in your Apple Music library with a single
command and **zero manual steps after that** — no drag-and-drop, no File → Add to
Library.

```
addsong <url>   →   (song appears in Apple Music a moment later)
```

## High-level flow

```
 ┌──────────┐    ┌─────────┐    ┌──────────────┐    ┌──────────────────────────┐
 │  addsong │ →  │ yt-dlp  │ →  │   ffmpeg     │ →  │  move into watch folder  │
 │  <url>   │    │download │    │ convert+tag  │    │  (Apple Music imports)   │
 └──────────┘    └─────────┘    └──────────────┘    └──────────────────────────┘
   you run it     gets audio     m4a + artwork        hands-off auto-import
```

## Components

### 1. `addsong` — the wrapper script
A small shell script (~15–25 lines) that ties everything together. Responsibilities:

- Validate it got a URL argument.
- Call `yt-dlp` with the right flags.
- Make sure the output lands in (or is moved to) the watch folder.
- Print clear success/failure output.

It deliberately stays thin — the heavy lifting is done by yt-dlp and ffmpeg, which
are battle-tested.

### 2. `yt-dlp` — the downloader
Handles fetching the best available audio stream from YouTube. Key flags the script
will rely on:

| Concern              | yt-dlp flag (conceptual)                                  |
|----------------------|-----------------------------------------------------------|
| audio only           | `-x` / `--extract-audio`                                  |
| target format        | `--audio-format m4a`                                       |
| embed cover art      | `--embed-thumbnail`                                        |
| embed metadata       | `--embed-metadata`                                         |
| parse title→fields   | `--parse-metadata` (to split "Artist - Title")            |
| output location/name | `-o` template                                             |

`yt-dlp` calls `ffmpeg` under the hood for conversion and embedding, which is why
both are required.

### 3. `ffmpeg` — conversion & tagging engine
Not called directly by us in the common path; yt-dlp invokes it. It converts the
downloaded stream to AAC in an `.m4a` container and embeds artwork + tags.

### 4. Apple Music watch folder — the import mechanism
The crux of the "instant" experience. Apple Music continuously watches:

```
~/Music/Music/Media.localized/Automatically Add to Music.localized/
```

Any audio file placed there is automatically imported into the library and the
original is filed into the Media folder. **We never script Apple Music itself** —
we just feed this folder. This is simpler and more robust than the AppleScript
alternative.

## Key design decisions

### Why a CLI command, not a curl-able server?
The first idea was `curl localhost/add?url=...` against a background daemon. A plain
CLI command (`addsong <url>`) does the same job with far less moving machinery —
no always-on process, no port, no web framework. Chosen for simplicity. A server
front-end could be added later as an optional layer if remote triggering is ever
wanted.

### Why the watch folder instead of AppleScript?
Two ways to get a file into Apple Music programmatically:
1. **Watch folder** — drop file, Apple imports it. Stateless, no app scripting.
2. **AppleScript** (`osascript` telling `Music.app` to add a file) — more control
   (e.g. add to a specific playlist) but more fragile and app-version-sensitive.

We use the watch folder as the default because it's the most reliable "hands-off"
path. AppleScript is a possible future enhancement (e.g. auto-add to a playlist).

### Why m4a (AAC) over mp3?
`m4a`/AAC is Apple's native format, integrates cleanly with Apple Music, and gives
better quality per bitrate than mp3. mp3 is only preferable if you need maximum
portability to non-Apple players — not the goal here.

## Edge cases & how we handle them

| Edge case | Plan |
|-----------|------|
| **Ugly titles** — `Artist - Song (Official Video) [4K]` | Use yt-dlp `--parse-metadata` + a cleanup pass to strip `(Official...)`, `[4K]`, `(Lyrics)`, etc. This is the single biggest quality lever. |
| **Artist/title split** | Prefer YouTube Music–style metadata when present; fall back to splitting on `" - "`. Imperfect; Phase 2 work. |
| **Playlists** | A playlist URL could grab every track. Default to **single track**; add an explicit `--playlist` flag to opt in (avoids accidental 200-song dumps). |
| **Duplicates** | Apple Music does **not** dedupe the watch folder — re-running on the same song creates a second copy. No automatic guard in v1; user beware. |
| **Download failure / bad URL** | Script should exit non-zero with a readable message instead of silently dropping nothing into the watch folder. |
| **Watch folder missing/renamed** | Path varies by macOS version and whether the library was renamed. Script should verify the folder exists and error clearly if not. See SETUP. |
| **Region-locked / age-gated videos** | yt-dlp may need cookies; out of scope for v1, document as a known limitation. |

## What this project is NOT

- It does **not** rip or remove DRM from Apple Music's own catalog — that's
  impossible by design and not a goal.
- It does **not** target Windows. The download/tag half is cross-platform, but the
  reliable hands-off auto-import depends on macOS Apple Music's watch folder
  (Windows' iTunes has an equivalent; the newer Apple Music for Windows app does
  not reliably). Mac-only keeps the design clean.

## Possible future extensions

- AppleScript layer to add tracks to a **specific playlist** on import.
- Optional **curl-able local server** front-end for remote/Shortcut triggering.
- A small **config file** for output format, library path, and default playlist.
- **Dedup check** against the library before importing.
