# ADR-0002 — gRPC bidirectional streaming rather than REST polling, WebSocket, or an RTP-style media transport

**Status:** Accepted (as built; recorded retrospectively 2026-08-08)

## Context

A session is a continuous, ordered, bidirectional exchange: capture chunks
flow up (audio every 125 ms, frames at 8–30 fps), results flow down at the
hop cadence (4–8 per second), and in touch mode control messages (queries,
freeze, sticky) ride the same ordered stream as capture. The contract must
be identical on a Python server and a Kotlin client, survive schema
evolution, and give the client a machine-readable failure taxonomy (model
not loaded, unreachable, mid-stream error).

## Alternatives considered

- **REST polling** — request/response per window: no server push, per-call
  connection and header overhead at 8 results/s, no ordering guarantee
  across calls, and the touch-mode in-band controls would need a second
  channel. Rejected.
- **WebSocket** — full-duplex, but untyped: framing, message schema, status
  codes, and codegen would all be hand-rolled twice (Python and Kotlin),
  recreating what gRPC/Protobuf provide. Rejected.
- **RTP/WebRTC-style media transport** — designed for continuous media, but
  the payloads here are model-shaped (JPEG pairs, PCM blocks, heatmaps,
  typed queries), not codec frames; would require custom payload formats, a
  separate signalling channel, and still no typed RPC error model. Loss
  tolerance — RTP's main gift — is wrong for the audio path, which must be
  lossless (FR-011). Rejected.

## Decision

One gRPC bidirectional stream per session (`StreamProcess`), Protobuf
messages (ADR-0003), model selection via call metadata, gRPC status codes
for failure semantics, over HTTP/2 on port 50051.

## Consequences

Positive:
- One ordered stream carries capture, results, and touch controls
  (FR-010, FR-016); per-message overhead is small at the required cadence
  (NFR-PERF-003, NFR-PERF-013).
- Generated stubs on both ends from one contract file (UC-21,
  NFR-COMPAT-001); metadata gives per-stream model selection (FR-012) and
  `FAILED_PRECONDITION` a precise meaning (FR-061).
- Deadlines, keepalive, and message-size caps come with the transport
  (FR-014, FR-015).

Negative:
- Toolchain weight: protoc + Gradle protobuf plugin + grpc-kotlin on the
  client; stub regeneration discipline across two repositories (UC-21 exists
  because of this).
- TCP head-of-line blocking under loss: one lost segment stalls the whole
  multiplexed stream — a real cost on poor Wi-Fi, unmeasured (E-N gap).
- No media clocking: pacing and underrun handling are the client's problem —
  the adaptive jitter buffer (FR-051) is this decision's bill.
- Plaintext configuration is explicit (`usePlaintext`), inseparable from the
  trusted-LAN posture (NFR-SEC-001; FR-P01).

## Requirements served

FR-010–FR-017, FR-061; NFR-PERF-003, NFR-PERF-013, NFR-COMPAT-001/002.
Costs land on: FR-051, UC-21, E-N characterisation, FR-P01.

## Evidence

`sonicsight.proto` (service + messages); `src/grpc_server.py:958-975`
(server options); `GrpcModule.kt:51-55` (client channel); TR §6.
