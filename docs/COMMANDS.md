# CopyCat Command Reference

A complete, example-driven reference for every `copycat` command and option. For
the raw auto-generated help, see [CLI_REFERENCE.md](CLI_REFERENCE.md); for a
narrative overview, see [USER_GUIDE.md](USER_GUIDE.md).

- [Conventions](#conventions)
- [Exit codes](#exit-codes)
- [Global options](#global-options)
- [`copy`](#copy) · [`paste`](#paste) · [`remote`](#remote) · [`config`](#config)
  · [`api`](#api) · [`install-gui`](#install-gui) · [`completions`](#completions)
  · [`man`](#man)

---

## Conventions

- An **endpoint** is a local path (`./dir`, `C:\data`) or a remote URI
  (`sftp://user@host:port/path`). No `scheme://` ⇒ local.
- Global options are declared **once** and apply wherever they're relevant; they
  may appear before or after the subcommand (they're "global"), so
  `copycat -r 3 copy a b` and `copycat copy a b -r 3` are equivalent.
- `SRC`/`DST` are positional; everything with `-`/`--` is an option.

## Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success — including `--help`, `--version`, and running `copycat` with no subcommand (prints help). |
| `1` | Runtime error (e.g. source missing, validation mismatch, remote unavailable). Message on stderr as `copycat: error: …`. |
| `2` | Usage error (unknown subcommand, missing/!invalid argument). `clap` prints usage to stderr. |

---

## Global options

These are accepted by the transfer commands (`copy`, `remote`); `paste` honors
the ones relevant to copying files. They layer **on top of the config file**
(CLI always wins).

| Option | Arg | Default | What it does |
|--------|-----|---------|--------------|
| `-r, --retry` | `N` | `2` (config) | Retry each **file** up to N extra times with exponential backoff before failing it. |
| `-e, --exclude` | `PATTERN,…` | config list | Skip matching entries. Bare names match at any depth (`node_modules`); globs supported (`*.tmp`); repeatable or comma-separated. |
| `--exclude-from` | `FILE` | – | Read extra patterns, one per line (`#` comments and blanks ignored). Merged with `-e`. |
| `--replace` | – | off | Overwrite an existing destination instead of renaming. Mutually exclusive with `--confirm`. |
| `--confirm` | – | off | Prompt (`y/N`) before overwriting; on a non-interactive stdin it declines and falls back to renaming. |
| `--dry-run` | – | off | Plan and report what would happen; write nothing. |
| `--resume` | – | off | Resume a partial destination: skip complete files, append the remainder of partial ones. Targets the original path (no rename). |
| `-j, --jobs` | `N` | `1` | Transfer up to N files concurrently. |
| `--limit-rate` | `BYTES` | unlimited | Cap throughput in bytes/second (e.g. `1048576` = 1 MiB/s). |
| `--identity` | `FILE` | – | SSH private key for remote auth. |
| `--agent` | – | off | Authenticate remotes via the SSH agent. |
| `-v, --verbose` | – | off | Debug-level logging to stderr (or set `RUST_LOG`). |
| `-h, --help` | – | – | Print help. `-V, --version` prints the version. |

> **Conflict precedence:** `--replace` → `--confirm` → config default (`rename`).
> **Auth precedence (remote):** `--identity` → `--agent` → `COPYCAT_PASSWORD`
> env var. Passwords are never taken as a flag and never written to disk.

---

## `copy`

Copy a file or directory between two endpoints (local or remote).

```
copycat [GLOBALS] copy <SRC> <DST>
```

**Behavior**
- `DST` doesn't exist → it becomes a copy of `SRC`.
- `DST` is an existing directory → `SRC` is copied **into** it under its basename
  (like `cp -r`).
- Copies stream with bounded memory and are validated (size + SHA-256 by default).
- `DST` of the form `buffer:NAME` stages the source into a named clipboard buffer
  (see [`paste`](#paste)).

**Examples**

```bash
# Simplest: one file
copycat copy report.pdf ~/backup/report.pdf

# Single option — recursive dir copy excluding one folder
copycat -e node_modules copy ./project ./backup

# Single option — preview only, change nothing
copycat --dry-run copy ./project ./backup

# Multiple options — retries, excludes, overwrite, 4 parallel files (chained)
copycat -r 3 -e node_modules,.git,"*.tmp" --replace -j 4 copy ./project ./backup

# Exclude list from a file + inline, with bandwidth cap
copycat --exclude-from .copycatignore -e .DS_Store --limit-rate 2097152 copy ./media /mnt/usb

# Resume an interrupted copy into the same destination
copycat --resume copy ./big-dataset /mnt/archive/big-dataset

# Stage into a named internal buffer (then paste elsewhere)
copycat copy ./release.zip buffer:work
```

---

## `paste`

Paste from the system clipboard or a named internal buffer.

```
copycat paste [DEST] [--buffer NAME]
```

- `DEST` defaults to `.` (current directory).
- **No `--buffer`:** reads **text** from the system clipboard. If `DEST` is `.`
  it prints to stdout; otherwise it writes the text to that file.
- **`--buffer NAME`:** if the buffer holds **files**, they are copied into `DEST`;
  if it holds **text**, it is written/printed as above.

**Examples**

```bash
# System clipboard text → stdout
copycat paste

# System clipboard text → a file
copycat paste notes.txt

# Files previously staged with `copy … buffer:work` → into ./incoming
copycat paste ./incoming --buffer work
```

---

## `remote`

Perform a transfer with one explicit local and one remote side. (Plain `copy`
also accepts remote endpoints; `remote` makes direction explicit.) Requires a
build with the `sftp` feature.

```
copycat [GLOBALS] remote push <LOCAL_SRC> <REMOTE_DST>
copycat [GLOBALS] remote pull <REMOTE_SRC> <LOCAL_DST>
```

**Examples**

```bash
# Upload a directory over SFTP, authenticating with a key
copycat --identity ~/.ssh/id_ed25519 remote push ./site sftp://deploy@host:22/var/www

# Download a file using the SSH agent, with 3 retries
copycat --agent -r 3 remote pull sftp://user@host/var/log/app.log ./app.log

# Password auth via env var (never a flag), excluding caches, resuming
export COPYCAT_PASSWORD='…'
copycat --resume -e "*.cache" remote push ./build sftp://user@host/opt/app
```

> Host keys are verified against `~/.ssh/known_hosts`.

---

## `config`

View and edit the JSON configuration (defaults that CLI flags override).

```
copycat config path                 # print the resolved config file path
copycat config init [--force]       # write a default config (––force overwrites)
copycat config get <KEY>            # print one value
copycat config set <KEY> <VALUE>    # set one value
```

**Keys:** `retries`, `excludes` (comma-separated), `conflict`
(`rename|replace|confirm`), `validation` (`size-sha256|size|none`),
`remote.username`, `remote.sftp_port`, `api.bind`, `api.port`.

**Examples**

```bash
copycat config path
copycat config init
copycat config set retries 3
copycat config set conflict replace
copycat config set excludes "node_modules,.git,target"
copycat config get validation
```

---

## `api`

Run the gRPC API server (`copycat.v1.CopyService`) over the same engine.

```
copycat api serve [--port N] [--bind IP] [--expose] [--token T]
                  [--tls-cert FILE --tls-key FILE]
                  [--max-concurrency N] [--metrics-port N]
```

| Option | Default | Notes |
|--------|---------|-------|
| `--port` | `50051` | gRPC port. |
| `--bind` | `127.0.0.1` | Only honored with `--expose`; otherwise loopback is forced. |
| `--expose` | off | Allow a non-loopback bind — **requires** `--token` or TLS. |
| `--token` | – | Bearer token required on every call (or `COPYCAT_API_TOKEN`). |
| `--tls-cert` / `--tls-key` | – | Enable TLS (both required together). |
| `--max-concurrency` | `64` | Max in-flight requests per connection. |
| `--metrics-port` | – | Also serve Prometheus metrics at `/metrics` on this port. |

**Examples**

```bash
# Local-only, default port
copycat api serve

# Different port + Prometheus metrics
copycat api serve --port 9000 --metrics-port 9090

# Exposed on the network, authenticated with a token
export COPYCAT_API_TOKEN="$(openssl rand -hex 16)"
copycat api serve --expose --bind 0.0.0.0 --token "$COPYCAT_API_TOKEN"

# Exposed over TLS
copycat api serve --expose --bind 0.0.0.0 --tls-cert cert.pem --tls-key key.pem
```

The server refuses to start exposed without a token or TLS (exit `1`).

---

## `excluder`

Find junk (by name or glob) under a location and **safely** remove it. Deletes
are reversible: matched items are **moved** into a quarantine directory (never
destroyed) with a manifest, and `undo` restores them.

```
copycat excluder find <ROOT> -p <patterns> [--json]
copycat excluder delete <ROOT> -p <patterns> [-y]
copycat excluder undo <SESSION_ID>
copycat excluder list
```

**Examples**

```bash
# See what would be removed (size + modified date), largest first
copycat excluder find ./my-project -p node_modules,*.log,.cache

# Quarantine it (prompts unless -y); prints an undo session id
copycat excluder delete ./my-project -p node_modules,dist,target -y

# Changed your mind — put everything back
copycat excluder undo <session-id>

# List sessions you can still undo
copycat excluder list
```

- Patterns match like excludes: a bare name (`node_modules`) matches at any
  depth; globs (`*.log`, `build/**`) are supported.
- A matched directory is reported/handled as one item (its contents aren't
  listed separately).
- `find --json` emits machine-readable output (path, size, created/modified).

---

## `install-gui`

Detect the platform package manager and print the command to install the
optional `copycat-gui`. If `copycat-gui` is already on `PATH`, it says so.

```
copycat install-gui
```

Recommends `brew install --cask copycat-gui` (macOS), `winget`/`choco` (Windows),
or `apt`/`dnf` (Linux), depending on what's available.

---

## `completions`

Generate a shell completion script to **stdout**.

```
copycat completions <bash|zsh|fish|powershell|elvish>
```

**Examples**

```bash
# bash (system-wide)
copycat completions bash | sudo tee /etc/bash_completion.d/copycat >/dev/null

# zsh (into a dir on your $fpath)
copycat completions zsh > ~/.zfunc/_copycat

# fish
copycat completions fish > ~/.config/fish/completions/copycat.fish

# PowerShell (append to your profile)
copycat completions powershell >> $PROFILE
```

---

## `man`

Render the man page (roff) to **stdout**.

```bash
copycat man > copycat.1
man ./copycat.1            # preview
sudo install -m644 copycat.1 /usr/share/man/man1/copycat.1   # install
```

---

_This document is maintained by hand to stay example-rich; the flag tables are
kept in sync with the `--help` output in [CLI_REFERENCE.md](CLI_REFERENCE.md)._
