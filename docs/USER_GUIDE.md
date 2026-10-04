# CopyCat User Guide

This guide covers everyday use of the `copycat` CLI for local and remote
transfers. For the exhaustive flag list see [CLI_REFERENCE.md](CLI_REFERENCE.md)
or run `copycat --help` / `copycat <command> --help`.

## Concepts

- **Endpoint** — a local path (`./dir`, `C:\data`) or a remote URI
  (`sftp://user@host:port/path`). Anything without a `scheme://` is local.
- **Job** — one `copy`/`remote` invocation. Jobs apply excludes, conflict
  policy, retries, and validation uniformly regardless of endpoint type.
- **Config** — JSON defaults layered under CLI flags (flags always win).

## Global options

| Flag | Meaning | Default |
|------|---------|---------|
| `-r, --retry N` | retries per file | `2` |
| `-e, --exclude P,…` | exclude patterns (globs or names) | config list |
| `--exclude-from FILE` | read extra patterns (one per line) | – |
| `--replace` | overwrite instead of renaming | off |
| `--confirm` | prompt before overwrite (TTY) | off |
| `--dry-run` | plan only, change nothing | off |
| `--resume` | resume partial destinations | off |
| `-j, --jobs N` | concurrent file transfers | `1` |
| `--limit-rate BYTES` | bandwidth cap (bytes/sec) | unlimited |
| `--identity FILE` | SSH private key | – |
| `--agent` | use the SSH agent | off |
| `-v, --verbose` | debug-level logging | off |

## Local transfers

```bash
copycat copy SRC DST
```

- If `DST` does not exist, it becomes a copy of `SRC`.
- If `DST` is an existing directory, `SRC` is copied **into** it under its
  basename (like `cp -r`).
- Copies are **streamed** (bounded memory) and **validated** (size + SHA-256 by
  default). A mismatch fails the file in strict mode.

### Excludes

Patterns without a `/` match that name at any depth (like `.gitignore`):

```bash
copycat -e node_modules,.git,"*.tmp" copy ./src ./dst
copycat --exclude-from .copycatignore copy ./src ./dst
```

Default excludes (from config): `.git`, `node_modules`, `__pycache__`, `.DS_Store`.

### Conflicts

Default is **rename** (`file (1).txt`, `folder (1)`). Override with `--replace`
or `--confirm`. The default and naming strategy are configurable.

## Remote transfers (SFTP)

Build with the `sftp` feature, then:

```bash
export COPYCAT_PASSWORD=secret           # or use --identity / --agent
copycat remote push ./local  sftp://user@host:22/remote/path
copycat remote pull sftp://user@host/remote/file ./local/file
copycat copy ./a sftp://user@host/b      # copy also accepts remote endpoints
```

- **Auth precedence**: key file (`--identity`) → agent (`--agent`) → password
  (`COPYCAT_PASSWORD`). Passwords are never read from a flag or stored on disk.
- **Host keys** are verified against `~/.ssh/known_hosts`.

## Resume

`--resume` targets the original destination (no renaming), skips files already
complete, and appends the remaining bytes of partial files. Safe to re-run the
same command after an interruption.

## Clipboard

```bash
copycat copy FILE buffer:NAME     # stage a file reference into a named buffer
copycat paste DEST --buffer NAME  # copy the buffered files into DEST
copycat paste DEST                # paste system-clipboard text (DEST=. → stdout)
```

Internal buffers persist across runs under the data directory.

## Configuration

```bash
copycat config path               # where the JSON config lives
copycat config init               # write defaults
copycat config get retries
copycat config set conflict replace
```

Keys: `retries`, `excludes`, `conflict`, `validation`, `remote.username`,
`remote.sftp_port`, `api.bind`, `api.port`.

## gRPC API

```bash
copycat api serve --port 50051                 # loopback only
copycat api serve --expose --bind 0.0.0.0 --token "$TOKEN"   # requires auth
copycat api serve --tls-cert cert.pem --tls-key key.pem
copycat api serve --metrics-port 9090          # Prometheus at /metrics
```

The service (`copycat.v1.CopyService`) exposes `StartCopy`, `WatchProgress`
(streaming), `GetStatus`, `Cancel`, and `GetInfo`. The server binds `127.0.0.1`
unless `--expose` is given, and refuses to expose without a token or TLS.

## Observability

- Logs go to stderr; set `RUST_LOG` or pass `--verbose`.
- Every operation appends a redacted JSON line to the audit log under the data
  directory.

## Exit codes

`0` success · `1` error (message on stderr as `copycat: error: …`).

## Shell completions & man page

```bash
copycat completions {bash|zsh|fish|powershell} > <target>
copycat man > copycat.1
```
