# Build from Source, Install & Configure

How to build the `copycat` CLI (and the optional GUI) yourself on **Windows**,
**macOS**, and **Linux (Ubuntu/Debian & Fedora/RHEL)**, install the executable
into your OS, put it on your `PATH`, configure it, and run it.

> Prefer a prebuilt package? See the [README](../README.md#install) and
> [packaging/PUBLISHING.md](../packaging/PUBLISHING.md). This guide is for building
> from source.

- [1. Prerequisites](#1-prerequisites)
- [2. Get the source](#2-get-the-source)
- [3. Build the CLI](#3-build-the-cli)
- [4. Install the CLI (place it + add to PATH)](#4-install-the-cli)
- [5. Shell completions & man page](#5-shell-completions--man-page)
- [6. Configure CopyCat](#6-configure-copycat)
- [7. Run it](#7-run-it)
- [8. Build & install the GUI (optional)](#8-build--install-the-gui-optional)
- [9. Update / uninstall](#9-update--uninstall)
- [10. Troubleshooting](#10-troubleshooting)

---

## 1. Prerequisites

All platforms need the **Rust toolchain** and **protoc** (the CLI links the gRPC
API crate, which is generated from a `.proto` at build time).

### Rust (every OS)

Install via [rustup](https://rustup.rs) (Rust ≥ 1.80):

```bash
# macOS / Linux
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

On **Windows**, download and run `rustup-init.exe` from <https://rustup.rs> (pick
the default `x86_64-pc-windows-msvc` toolchain).

### Per-OS build tools + protoc

**Windows**
- **Visual Studio Build Tools** with *"Desktop development with C++"* (provides the
  MSVC linker). Install from the Visual Studio installer.
- **protoc**: `choco install protoc` (or `scoop install protoc`, or download from
  the [protobuf releases](https://github.com/protocolbuffers/protobuf/releases) and
  add it to `PATH`).

**macOS**
```bash
xcode-select --install          # C toolchain
brew install protobuf           # protoc
```

**Ubuntu / Debian**
```bash
sudo apt update
sudo apt install -y build-essential pkg-config protobuf-compiler
```

**Fedora / RHEL / CentOS / Rocky / Alma**
```bash
sudo dnf install -y gcc gcc-c++ make pkgconf-pkg-config protobuf-compiler
```

> The optional **SFTP** feature uses pure-Rust `russh` with the `ring` backend —
> it just needs the C toolchain above (no extra system libraries, no OpenSSL).

---

## 2. Get the source

```bash
git clone https://github.com/wirekind/copycat.git
cd copycat
```

---

## 3. Build the CLI

The Rust workspace lives in `cli/`.

```bash
cd cli

# Local-only build (no remote transport)
cargo build --release

# …or include the SFTP remote transport
cargo build --release --features sftp
```

The optimized binary is written to:

| OS | Path |
|----|------|
| Windows | `cli\target\release\copycat.exe` |
| macOS / Linux | `cli/target/release/copycat` |

Quick sanity check:

```bash
./target/release/copycat --version
```

---

## 4. Install the CLI

### Option A — `cargo install` (recommended, cross-platform)

Builds and copies `copycat` into Cargo's bin dir (`~/.cargo/bin`, or
`%USERPROFILE%\.cargo\bin` on Windows), which **rustup already adds to your
`PATH`** — so it just works afterward:

```bash
# from the repo root
cargo install --path cli/crates/copycat-cli                 # local-only
cargo install --path cli/crates/copycat-cli --features sftp # with SFTP
```

Then open a new terminal and run `copycat --version`.

### Option B — place the binary manually

If you'd rather control where it lives:

**Windows** *Powershell (as Administrator) from "copycat" root folder*
```powershell
# Choose a folder, e.g. a personal tools dir:
$dir = "$env:LOCALAPPDATA\Programs\CopyCat"
New-Item -ItemType Directory -Force $dir | Out-Null
Copy-Item cli\target\release\copycat.exe $dir\

# Add it to your user PATH (persists; open a NEW terminal afterward):
[Environment]::SetEnvironmentVariable(
  "Path",
  [Environment]::GetEnvironmentVariable("Path","User") + ";$dir",
  "User")
```
(Or run `setx PATH "%PATH%;%LOCALAPPDATA%\Programs\CopyCat"`, or add the folder via
*System Properties → Environment Variables → Path → Edit*.)

**macOS / Linux**
```bash
# System-wide (needs sudo) — already on PATH:
sudo install -m755 cli/target/release/copycat /usr/local/bin/copycat

# …or per-user (no sudo):
mkdir -p ~/.local/bin
install -m755 cli/target/release/copycat ~/.local/bin/copycat
# Ensure ~/.local/bin is on PATH (add to ~/.bashrc or ~/.zshrc if needed):
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

Verify from a fresh shell:
```bash
copycat --version
```

---

## 5. Shell completions & man page

CopyCat generates these itself; install the ones you want.

```bash
# Bash (Linux)
copycat completions bash | sudo tee /etc/bash_completion.d/copycat >/dev/null

# Zsh (place on your $fpath)
copycat completions zsh > ~/.zfunc/_copycat     # ensure: fpath+=~/.zfunc; autoload -U compinit; compinit

# Fish
copycat completions fish > ~/.config/fish/completions/copycat.fish

# PowerShell (append to your profile)
copycat completions powershell >> $PROFILE

# Man page (Linux/macOS)
copycat man | sudo tee /usr/share/man/man1/copycat.1 >/dev/null
```

---

## 6. Configure CopyCat

CopyCat reads a JSON config, and CLI flags always override it. Find/initialize it:

```bash
copycat config path     # prints the exact config file location for your OS
copycat config init     # writes a default config there
```

**Default locations** (run `copycat config path` for the authoritative value):

| OS | Config file | Data (audit log, clipboard buffers) |
|----|-------------|--------------------------------------|
| Windows | `%APPDATA%\Wirekind\CopyCat\config\config.json` | `%APPDATA%\Wirekind\CopyCat\data\` |
| macOS | `~/Library/Application Support/io.Wirekind.CopyCat/config.json` | same folder |
| Linux | `~/.config/copycat/config.json` | `~/.local/share/copycat/` |

Edit values from the CLI (or edit the JSON directly):

```bash
copycat config set retries 3
copycat config set conflict replace          # rename | replace | confirm
copycat config set validation size-sha256    # size-sha256 | size | none
copycat config set excludes "node_modules,.git,target"
copycat config get retries
```

### Environment variables

| Variable | Used by | Purpose |
|----------|---------|---------|
| `COPYCAT_PASSWORD` | `copy`/`remote` (SFTP) | Remote password (never passed as a flag or stored). |
| `COPYCAT_API_TOKEN` | `copycat api serve` | Bearer token for the gRPC API. |
| `RUST_LOG` | all | Log verbosity, e.g. `RUST_LOG=debug` (or use `--verbose`). |

Set them the usual way:
```bash
# macOS / Linux
export COPYCAT_PASSWORD='…'
# Windows (PowerShell, current session)
$env:COPYCAT_PASSWORD = '…'
# Windows (persist for your user)
setx COPYCAT_PASSWORD "…"
```

---

## 7. Run it

```bash
copycat --help                                   # overview; exits 0
copycat copy ./project ./backup                  # local copy
copycat -j 4 -e node_modules,.git copy ./a ./b   # excludes + parallelism
copycat --dry-run copy ./a ./b                   # preview, change nothing
copycat config path
```

See the [Command Reference](COMMANDS.md) for every command with examples, and the
[User Guide](USER_GUIDE.md) for a narrative walkthrough (including remote/SFTP).

---

## 8. Build & install the GUI (optional)

The GUI is a separate Tauri project in `gui/`. It needs extra system libraries.

**Extra prerequisites**
- **Windows**: the **WebView2 Runtime** (preinstalled on Windows 10/11) + the MSVC
  build tools from §1.
- **macOS**: Xcode Command Line Tools (from §1).
- **Ubuntu/Debian**:
  ```bash
  sudo apt install -y libwebkit2gtk-4.1-dev libgtk-3-dev librsvg2-dev
  ```
- **Fedora/RHEL**:
  ```bash
  sudo dnf install -y webkit2gtk4.1-devel gtk3-devel librsvg2-devel
  ```

**Build**
```bash
cd gui

# Run it directly (dev):
cargo run -p copycat-gui

# …or produce native installers (.msi/.exe, .dmg, .deb/.AppImage):
cargo install tauri-cli --version "^2"   # one-time
cargo tauri build                        # output under gui/src-tauri/target/release/bundle/
```

Install the produced installer like any app for your OS. From the CLI,
`copycat install-gui` prints the package-manager command for your platform.

---

## 9. Update / uninstall

**Update**: `git pull`, then re-run the build/install step from §3–§4.

**Uninstall**
```bash
# If installed via cargo:
cargo uninstall copycat-cli

# If placed manually:
rm /usr/local/bin/copycat          # or ~/.local/bin/copycat
# Windows: delete the copied copycat.exe and remove its folder from PATH

# Remove config/data (optional):
#   delete the folders shown by `copycat config path` (and the sibling data dir)
```

---

## 10. Troubleshooting

- **`protoc` not found / failed to compile `copycat-proto`** — install protoc
  (§1) and ensure it's on `PATH` (`protoc --version`), then rebuild.
- **`link.exe not found` (Windows)** — install the Visual Studio Build Tools with
  *Desktop development with C++*, then reopen the terminal.
- **`copycat: command not found` after install** — open a **new** terminal (PATH
  changes don't affect already-open shells); confirm the install dir is on `PATH`.
- **GUI fails to build on Linux** — install the WebKitGTK/GTK packages in §8.
- **SFTP build issues** — ensure a C compiler is present (§1); the SFTP feature
  uses the `ring` backend and needs no OpenSSL/NASM.
- **Need more logs** — run with `--verbose` or `RUST_LOG=debug`.
