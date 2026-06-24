# Architecture

`addsong` is a single Bash script. It has no daemon, no API access, and no
dependency on AppleScript or the Music app's scripting interface. It relies on
one macOS behavior: Apple Music automatically imports any audio file dropped
into its "Automatically Add to Music" folder.

## The pipeline (per track)

1. **Read metadata, no download.** `yt-dlp --print` fetches the video id, title,
   and uploader without downloading anything.
2. **Clean it up.** `clean_meta()` strips junk like `(Official Video)`, `[4K]`,
   and `(feat. X)`. If the title looks like `Artist - Title` it is split;
   otherwise the uploader becomes the artist.
3. **Review (interactive runs).** For a single track at a terminal, the resolved
   artist/title are shown so you can accept, edit, or skip before any download.
   Playlists are non-interactive unless `--edit` is passed.
4. **Duplicate guard.** Each imported track's video id is recorded in a ledger
   (`ADDSONG_LEDGER`). Already-seen ids are skipped unless `--force` is given.
5. **Download + tag.** `yt-dlp` extracts the audio and embeds the thumbnail;
   `ffmpeg` writes the chosen title/artist tags (copying streams, so the
   embedded artwork is preserved).
6. **Hand off to Apple Music.** The tagged file is moved into the watch folder.
   Apple Music imports it on its own a moment later.

## Why the watch folder

Using the watch folder instead of AppleScript or the Music API means:

- **No credentials.** No Apple ID, API key, or cookies are involved.
- **Fewer moving parts.** Importing is the OS's job; the script just produces a
  correctly tagged file and moves it.
- **Resilience.** If Music is closed when a file is written, it is imported the
  next time Music opens.

## Key components in the script

| Function        | Responsibility                                             |
|-----------------|------------------------------------------------------------|
| `clean_meta()`  | Normalize a metadata string (strip junk, collapse spaces). |
| `safe_name()`   | Make a string safe to use as a filename.                   |
| `ledger_has()` / `ledger_add()` | Duplicate detection by video id.           |
| `review_meta()` | Interactive accept/edit/skip prompt (reads `/dev/tty`).    |
| `process_one()` | Runs the full pipeline for one URL.                        |

## Exit codes (per track)

`process_one()` returns `0` (added), `2` (skipped — duplicate or user skip), or
`1` (failed). The top-level run aggregates these into the end-of-run summary and
exits non-zero if any track failed.

## State and side effects

- **Ledger:** append-only TSV at `ADDSONG_LEDGER`
  (`~/.local/state/addsong/imported.tsv`), one row per imported track.
- **Staging:** each download uses a `mktemp -d` directory that is removed
  whether the track succeeds or fails.
- **Output:** exactly one tagged audio file moved into `ADDSONG_WATCH_DIR`.
