# QRun-IO Homebrew tap

Homebrew formulae for QRun-IO command-line tools. Today that is `qqq`: the QQQ command-line client
and its terminal UI (`qqq ui`) in one prebuilt binary, from the `cli-v*` GitHub releases of
[QRun-IO/qqq-cli](https://github.com/QRun-IO/qqq-cli).

| Formula | What | Platforms |
|---|---|---|
| `qqq` | QQQ command-line client and terminal UI, one binary | macOS arm64 and x64, Linux arm64 and x64 (glibc) |

## Install

```sh
brew install qrun-io/tap/qqq        # taps QRun-IO/homebrew-tap on first use
qqq --version                       # qqq 1.0.0 (... binary with ui ...)
qqq login --server https://your-qqq-server
qqq ui                              # the terminal UI, same binary
```

The formula installs `bin/qqq` and bash, zsh and fish completions. Nothing else.

## Upgrade

```sh
brew update
brew upgrade qrun-io/tap/qqq
```

## Notes

- The macOS binaries are signed ad hoc, not notarized. Homebrew does not set the quarantine flag, so
  Gatekeeper does not block them. A binary downloaded from the releases page with a browser is a
  different story; see the qqq-cli packaging notes.
- Alpine and other musl systems are not covered by Homebrew. Use the curl installer from qqq-cli,
  which picks the `-musl` build.
- Windows: use the Scoop bucket, [QRun-IO/scoop-bucket](https://github.com/QRun-IO/scoop-bucket).
- `qctl` lives in [QRun-IO/homebrew-qctl](https://github.com/QRun-IO/homebrew-qctl) for now.

## How the formula is updated

`Formula/qqq.rb` is generated in qqq-cli, at `docs/packaging/homebrew-tap/Formula/qqq.rb`; the copy here
should only differ from that file by version and sums. After a `cli-v<version>` GitHub release, in a
qqq-cli checkout:

```sh
gh release download cli-v<version> --repo QRun-IO/qqq-cli -p SHA256SUMS -D /tmp/qqq-<version>
node scripts/update-packaging.mjs /tmp/qqq-<version>/SHA256SUMS
cp docs/packaging/homebrew-tap/Formula/qqq.rb <this repo>/Formula/qqq.rb
```

`update-packaging.mjs` takes the version from `packages/cli/package.json` and puts each binary's sum from
the release's `SHA256SUMS` into the matching `url`/`sha256` pair. Always use the uploaded `SHA256SUMS`,
never a local rebuild: each build stamps its commit, so the sums differ. Open a pull request here with
the result.

## How CI verifies it

CircleCI runs the qqq-orb `homebrew-tap` profile (`.qqq-ci.yml`, orb track `0.8`; `.circleci/config.yml`
is the orb's canonical setup caller and must not be edited by hand). On every branch push, one job in a
digest-pinned `homebrew/brew` Linux container:

1. taps this exact checkout as `qrun-io/tap` and checks the tapped revision is the commit under test;
2. `brew readall --syntax --os=all --arch=all` (every OS and architecture block parses);
3. `brew fetch --all-platforms` (downloads all four binaries and checks every `sha256`);
4. `brew install` and `brew test` on Linux x64. The test runs `qqq --version` and requires
   `binary with ui`, and checks that shell completion answers.

The macOS binaries are downloaded and checksum-verified but not executed in CI.

## License

Apache-2.0, see [LICENSE](LICENSE). The formula is packaging only; qqq itself is licensed as stated in
qqq-cli.
