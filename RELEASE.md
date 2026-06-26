# Releasing addsong + publishing the Homebrew tap

The main repo is already pushed to https://github.com/ado11231/apple-music-pipeline.

## 1. Tag and push the release

The version in the script (`VERSION` in `addsong`) and the tag must match.
It is currently `1.0.0`.

```bash
git tag -a v1.0.0 -m "addsong 1.0.0"
git push origin v1.0.0
```

GitHub now serves a source tarball at:

```
https://github.com/ado11231/apple-music-pipeline/archive/refs/tags/v1.0.0.tar.gz
```

## 2. Compute the tarball checksum

```bash
curl -sL https://github.com/ado11231/apple-music-pipeline/archive/refs/tags/v1.0.0.tar.gz \
  | shasum -a 256
```

Copy the 64-character hash.

## 3. Create your tap repo (one time, reusable for every future tool)

Create an empty repo named exactly **`homebrew-tap`** on GitHub (no README, no
.gitignore), then clone it and add the formula:

```bash
git clone https://github.com/ado11231/homebrew-tap.git ~/homebrew-tap
mkdir -p ~/homebrew-tap/Formula
cp Formula/addsong.rb ~/homebrew-tap/Formula/addsong.rb
```

## 4. Fill in the sha256

In `~/homebrew-tap/Formula/addsong.rb`, replace the placeholder sha256 with the
hash from step 2:

```bash
sed -i 's/0000000000000000000000000000000000000000000000000000000000000000/PASTE_SHA256_HERE/' ~/homebrew-tap/Formula/addsong.rb
```

Commit and push:

```bash
cd ~/homebrew-tap
git add Formula/addsong.rb
git commit -m "addsong 1.0.0"
git push
```

## 5. Test the install

```bash
brew install ado11231/tap/addsong
addsong --version            # => addsong 1.0.0
```

Optionally audit the formula before publishing:

```bash
brew audit --new --formula ado11231/tap/addsong
brew test addsong
```

## Future releases

1. Bump `VERSION` in `addsong`, commit and push.
2. Tag `vX.Y.Z`, push the tag.
3. Recompute the sha256 (step 2) and update `url` + `sha256` in the tap's
   `Formula/addsong.rb`, then push the tap.

## Adding more tools to the same tap

Drop another `Formula/<tool>.rb` into the same `homebrew-tap` repo. Users get it
with `brew install ado11231/tap/<tool>` — no new tap needed.
