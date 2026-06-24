# Setup

`addsong` is macOS-only. It needs two command-line tools and a working Apple
Music watch folder.

## Step 1 — Install dependencies

Install [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) (downloads the audio) and
[`ffmpeg`](https://ffmpeg.org/) (converts and tags it). With
[Homebrew](https://brew.sh):

```bash
brew install yt-dlp ffmpeg
```

Keep `yt-dlp` current — an outdated version is the most common cause of failed
downloads:

```bash
brew upgrade yt-dlp
```

## Step 2 — Confirm the Apple Music watch folder

Apple Music auto-imports anything dropped into its "Automatically Add to Music"
folder. By default that is:

```
~/Music/Music/Media.localized/Automatically Add to Music.localized/
```

To verify it works, open the **Music** app and drag any `.m4a` or `.mp3` into
that folder. It should appear in your library within a couple of seconds.

If your library lives elsewhere, find the path under **Music > Settings > Files**
("Music Media folder location"), then point `addsong` at the *Automatically Add
to Music* folder inside it:

```bash
export ADDSONG_WATCH_DIR="/path/to/Automatically Add to Music.localized"
```

The first time `addsong` writes there, macOS may ask your terminal for
permission to access the Music folder. Click **Allow**.

## Step 3 — Install the command

Put `addsong` on your `PATH`:

```bash
mkdir -p ~/bin
mv addsong ~/bin/addsong
chmod +x ~/bin/addsong
```

If `~/bin` isn't on your `PATH`, add it (then open a new terminal):

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
```

Confirm it's installed:

```bash
addsong --version
```

## Configuration

All settings are environment variables (see `addsong --help`):

| Variable               | Purpose                       | Default                               |
|------------------------|-------------------------------|---------------------------------------|
| `ADDSONG_WATCH_DIR`    | Apple Music watch folder      | standard macOS path                   |
| `ADDSONG_AUDIO_FORMAT` | Output audio format           | `m4a`                                 |
| `ADDSONG_LEDGER`       | Imported-tracks list (dedup)  | `~/.local/state/addsong/imported.tsv` |
