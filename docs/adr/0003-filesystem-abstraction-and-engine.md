# 3. Unified Filesystem abstraction and streaming engine

Date: 2026-10-02

## Status

Accepted

## Context

CopyCat must copy between any combination of local and remote endpoints
(local→remote, remote→local, remote→remote) while applying excludes, conflict
handling, retries, and validation identically. An early byte-oriented
`Transport` trait (read/write whole blobs) was too coarse for large files and
recursive trees.

## Decision

- Define a `Filesystem` backend trait (classify, walk, streams, metadata,
  symlinks). `LocalFs`, an in-memory `MemoryFs` (for tests), and `SftpFs`
  implement it.
- A single `TransferEngine` drives a copy between any two `Filesystem` backends,
  streaming with bounded memory and emitting progress events.
- **Resume** is based on inspecting the destination (its partial bytes are the
  state), so it needs no separate journal and survives crashes.
- **Validation** fingerprints both sides through the backends' read streams, so
  it works the same for local and remote.

## Consequences

- New protocols only implement `Filesystem`; orchestration is untouched.
- Tests exercise the real engine code paths against `MemoryFs` as a stand-in for
  a remote server, so most remote behavior is covered without a network.
- Resume assumes a shorter destination is a valid prefix of the source (true for
  append-style interruptions).
