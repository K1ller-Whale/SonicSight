# ADR-0003 — Protocol Buffers rather than JSON for binary payloads

**Status:** Accepted (as built; recorded retrospectively 2026-08-08)

## Context

Every chunk and result is mostly binary: PCM16 audio blocks (2 756 bytes per
125 ms at 11 025 Hz), JPEG frames, uint8 heatmaps (3 136 bytes per side),
energy maps. A halves-mode result is ≈ 11.8 kB before compression at 8
results/s (TR §9.5). The contract must be consumed by generated code in
Python and Kotlin and must evolve additively (fields 8, 9, 15–24 were added
across two integrations without breaking older clients).

## Alternatives considered

- **JSON** — binary fields require base64 (+33 % size on every audio, JPEG,
  and map payload) plus per-message string parsing at 16 messages/s uplink;
  no schema, no codegen, contract drift policed by hand. Rejected.
- **MessagePack / CBOR** — binary-clean and compact, but schemaless: no
  generated types, no field-numbering discipline, and no native gRPC
  integration. Rejected.
- **FlatBuffers** — zero-copy reads, but marginal benefit at these message
  sizes, no first-class gRPC/Kotlin path in this stack, and a churn cost on
  two working codebases. Rejected.

## Decision

Protocol Buffers (proto3) as the single contract, kept as byte-identical
copies in both repositories, compiled by `grpc_tools.protoc` (backend) and
the Gradle protobuf plugin with the `lite` runtime (client).

## Consequences

Positive:
- Binary payloads ride untranslated; message sizes stay at the wire figures
  the throughput estimates assume (NFR-PERF-003, NFR-PERF-013).
- One typed contract, two generated stubs (UC-21); byte-identity is
  mechanically checkable (NFR-COMPAT-001); unknown-field semantics give
  additive evolution (FR-012's no-metadata default is one instance).

Negative:
- Two copies of the contract must be kept identical by discipline — the
  check exists precisely because drift is the failure mode.
- The `lite` runtime on Android drops reflection: no generic message
  debugging on-device.
- Comments in the proto are documentation, not code: the TR records seven
  places where proto comments and code disagree (TR §6.4, Table 10) — a
  maintenance cost this choice does not remove.

## Requirements served

NFR-COMPAT-001, NFR-PERF-003, NFR-PERF-013; UC-21; FR-012, FR-017.
Costs land on: NFR-COMPAT-001 (the mechanical drift check exists because of
this decision) and UC-21 (the regeneration discipline).

## Evidence

`sonicsight.proto`; byte-identity probe 2026-08-08 (ANALYSIS_REPORT
Appendix D); TR §6.2, §6.4, §9.5; `app/build.gradle.kts:87-119`.
