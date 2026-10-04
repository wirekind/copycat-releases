# 2. Language and frameworks

Date: 2026-10-02

## Status

Accepted

## Context

CopyCat needs a fast, memory-safe, cross-platform core shared by a CLI, a gRPC
API, and a GUI, producing small self-contained binaries.

## Decision

- **Rust** for all core/CLI/API code, with **Tokio** for async I/O and
  concurrency.
- **clap** (derive) for the CLI surface; **clap_complete** / **clap_mangen** for
  completions and man pages.
- **tonic** + **prost** for the gRPC API; API versioned via the proto package
  (`copycat.v1`).
- **Tauri** for the GUI (Rust backend + web frontend), reusing `copycat-core`.
- **JSON** configuration (via `serde`), in platform-standard locations.
- **Size + SHA-256** as the default post-transfer validation.
- **SFTP** (via pure-Rust `russh`) as the primary remote protocol, behind an
  optional `sftp` build feature; SCP/FTPS reserved for later.

## Consequences

- One engine, one set of behaviors across CLI/API/GUI.
- The SFTP crypto stack is heavier and needs a native assembler to build, so it
  is feature-gated to keep default builds light.
- `protoc` is required to build the proto crate.
