# XNU for davinci / sweet

The Mi 9T (`davinci`) reuses the SWEET machine configuration because both
targets use the SM7150-family XNU bring-up code. Davinci-specific board
identity and hardware support are supplied by the generated ADT and later
patches; the `kernel.development.sweet` artifact name is expected.

Built on GitHub's macOS arm64 runners by `.github/workflows/xnu.yml`, using
[blacktop/darwin-xnu-build](https://github.com/blacktop/darwin-xnu-build) (MIT) at a pinned commit.

- `MACOS_VERSION=13.5` -> xnu-8796 (same generation as iOS 16.6/16.7, the last iOS for iPhone 8).
- `patches/*.patch` are copied into the build script's patch dir and applied to the xnu tree with `git apply`.
- `python tools/reconstruct-file.py <path> <out>` replays the series and writes one
  resulting file, so a changed `.cpp` can be read or edited although the repo
  only carries patches.
- Output: `kernel.<config>.sweet` as a workflow artifact.

Fetch a build from this repository's workflow run:
`gh run download RUN_ID -R OWNER/ios-davinci -n xnu-kernel -D xnu/out`
