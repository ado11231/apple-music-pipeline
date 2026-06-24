# Releasing addsong + publishing the Homebrew tap

Everything here is yours to run when you're ready to publish. Nothing in this
repo has been pushed or tagged yet.

Throughout, replace `YOURNAME` with your GitHub username.

## 1. Push this repo to GitHub

```bash
git remote add origin https://github.com/YOURNAME/apple-music-pipeline.git
git push -u origin main
```

## 2. Tag and push the release

The version in the script (`VERSION` in `addsong`) and the tag must match.
It is currently `1.0.0`.

```bash
git tag -a v1.0.0 -m "addsong 1.0.0"
git push origin v1.0.0
```

GitHub now serves a source tarball at:

```
https://github.com/YOURNAME/apple-music-pipeline/archive/refs/tags/v1.0.0.tar.gz
```

## 3. Compute the tarball checksum

```bash
curl -sL https://github.com/YOURNAME/apple-music-pipeline/archive/refs/tags/v1.0.0.tar.gz \
  | shasum -a 256
```

Copy the 64-character hash.

## 4. Create your tap repo (one time, reusable for every future tool)

A tap is a GitHub repo named `homebrew-tap`. The `tap` part is what users type;
the repo can hold one formula per tool.

```bash
# create an empty repo named exactly "homebrew-tap" on GitHub, then:
git clone https://github.com/YOURNAME/homebrew-tap.git
mkdir -p homebrew-tap/Formula
cp Formula/addsong.rb homebrew-tap/Formula/addsong.rb
```

## 5. Fill in the formula

In `homebrew-tap/Formula/addsong.rb`:

- Replace every `YOURNAME` with your GitHub username.
- Replace the `sha256 "0000…"` placeholder with the hash from step 3.

A quick way (from inside the tap repo):

```bash
sed -i '' 's/YOURNAME/your-actual-username/g' Formula/addsong.rb
sed -i '' 's/0000000000000000000000000000000000000000000000000000000000000000/PASTE_SHA256_HERE/' Formula/addsong.rb
```

Commit and push:

```bash
git add Formula/addsong.rb
git commit -m "addsong 1.0.0"
git push
```

## 6. Test the install

```bash
brew install YOURNAME/tap/addsong
addsong --version            # => addsong 1.0.0
```

Optionally audit the formula before publishing:

```bash
brew audit --new --formula YOURNAME/tap/addsong
brew test addsong
```

## Future releases

1. Bump `VERSION` in `addsong`, commit.
2. Tag `vX.Y.Z`, push the tag.
3. Recompute the sha256 (step 3) and update `url` + `sha256` in the tap's
   `Formula/addsong.rb`, then push the tap.

## Adding more tools to the same tap

Drop another `Formula/<tool>.rb` into the same `homebrew-tap` repo. Users get it
with `brew install YOURNAME/tap/<tool>` — no new tap needed.
