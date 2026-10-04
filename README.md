<img src="assets/icons/paws_128.png" alt="CopyCat logo" width="96" align="right" />

# CopyCat

Cross-platform copy/paste utility for **local and remote** file transfers, with a
CLI (default) and an optional GUI. Built in Rust by **Wirekind Communications LLP**.

- **Local & remote**: recursive file/dir copy locally or over SFTP, in any
  direction (local↔remote, remote↔remote).
- **Safe by default**: rename-on-conflict, size + SHA-256 validation, retries
  with backoff, resume of interrupted transfers.
- **Scriptable**: excludes, dry-run, progress, shell completions, man page.
- **Programmable**: a gRPC API (`copycat api serve`) over the same engine.
- **Clipboard**: system clipboard + persistent named internal buffers.

> Status: pre-1.0. See [CHANGELOG.md](CHANGELOG.md) and
> [IMPLEMENTATION_TODO.md](IMPLEMENTATION_TODO.md) for progress.

## Install

CopyCat ships as a single `copycat` binary. Once published, package-manager
installs are:

```bash
# macOS / Linux (Homebrew)
brew install wirekind/tap/copycat

# Windows
winget install Wirekind.CopyCat     # or: choco install copycat

# Debian / Ubuntu (apt)
sudo apt install copycat            # from the CopyCat apt repo

# Fedora / RHEL / CentOS / Rocky / Alma (dnf)
sudo dnf install copycat            # from the CopyCat dnf repo
#  …or install a downloaded .rpm directly:
sudo dnf install ./copycat-<version>.x86_64.rpm

# npm / yarn (binary wrapper)
npm install -g @wirekind/copycat
```

The `.deb`/`.rpm` packages are produced by [nfpm](packaging/nfpm.yaml) in the
[release workflow](.github/workflows/release.yml); see
[packaging/README.md](packaging/README.md) for details.

### From source

```bash
cd cli
cargo build --release                 # binary at cli/target/release/copycat
cargo build --release --features sftp # …with the SFTP remote transport
```

For a full walkthrough — per-OS prerequisites (Windows/macOS/Ubuntu/Fedora),
installing the binary, adding it to `PATH`, configuration, and building the GUI —
see **[docs/BUILD_FROM_SOURCE.md](docs/BUILD_FROM_SOURCE.md)**.

## Quick start

```bash
# Local copy (recursive), excluding build junk, 4 parallel files
copycat -j 4 -e node_modules,.git copy ./project ./backup

# Dry run — show what would happen, change nothing
copycat --dry-run copy ./project ./backup

# Overwrite instead of renaming on conflict
copycat --replace copy a.txt b.txt

# Remote upload / download over SFTP (build with the sftp feature)
export COPYCAT_PASSWORD=...            # password never passed as a flag
copycat remote push ./site sftp://user@host:22/var/www
copycat remote pull sftp://user@host/var/log/app.log ./app.log

# Resume an interrupted transfer
copycat --resume copy ./big-dir sftp://user@host/backup

# Clipboard
copycat copy ./report.pdf buffer:work   # stage into a named buffer
copycat paste ./out --buffer work       # paste the buffer's files into ./out

# Configuration
copycat config path
copycat config set retries 3
copycat config get retries

# Run the gRPC API (loopback only by default)
copycat api serve --port 50051

# Shell completions / man page
copycat completions bash > /etc/bash_completion.d/copycat
copycat man > copycat.1
```

See the full [User Guide](docs/USER_GUIDE.md), the example-driven
[Command Reference](docs/COMMANDS.md), the generated
[CLI reference](docs/CLI_REFERENCE.md), and
[Build from Source & Install](docs/BUILD_FROM_SOURCE.md).

Running `copycat` with no subcommand prints help and exits `0`; usage errors exit
`2`; runtime errors exit `1`.

## Platforms

Windows (x64/arm64), macOS (Intel/Apple Silicon), Linux (Debian/Ubuntu and
Fedora/RHEL families).

## Project layout

```
cli/    Rust workspace: copycat-core, copycat-cli, copycat-proto, copycat-api
gui/    Tauri GUI (copycat-gui), consumes copycat-core
docs/   user guide, CLI reference, ADRs
packaging/  Homebrew, Winget, Chocolatey, deb/rpm (nfpm), npm wrapper
```

## Contributing & security

See [CONTRIBUTING.md](CONTRIBUTING.md) (tests are mandatory) and
[SECURITY.md](SECURITY.md). Licensed under [MIT](LICENSE).
