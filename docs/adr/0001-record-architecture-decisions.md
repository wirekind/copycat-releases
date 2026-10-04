# 1. Record architecture decisions

Date: 2026-10-02

## Status

Accepted

## Context

CopyCat spans a core library, CLI, gRPC API, and GUI, with several locked
technical decisions (see [OBJECTIVE_TODO.md](../../OBJECTIVE_TODO.md) §9). We want
a durable record of *why* significant choices were made, not just *what* the code
does.

## Decision

We keep lightweight Architecture Decision Records (ADRs) in `docs/adr/`, one file
per decision, numbered sequentially, using the Nygard format
(Context / Decision / Consequences). An ADR is added whenever a choice is
expensive to reverse or affects multiple components.

## Consequences

- New contributors can read the ADRs to understand the shape of the system.
- Superseded decisions are kept (marked `Superseded by NNNN`) rather than deleted,
  preserving history.
