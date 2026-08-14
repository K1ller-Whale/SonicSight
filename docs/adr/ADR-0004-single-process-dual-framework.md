# ADR-0004 — One process hosting both model frameworks rather than separate services

**Status:** Accepted (as built; recorded retrospectively 2026-08-08).
Supersedes the abandoned sidecar design (separate CAM and separation
services on ports 5556/5557), which was deleted before implementation
(HANDOFF §3.1, §9.5).

## Context

TensorFlow and PyTorch each allocate their own CUDA context and cuDNN kernel
images — roughly 1–1.3 GB of pure overhead per framework per process (TR
§9.4). In one process that overhead is paid once; across two processes,
twice. On the 6 GB measurement-environment card, that difference is the
difference between both models fitting and not fitting. Model switching must
also be fast: reloading a checkpoint on switch would cost the ≈ 4.2–4.5 s
cuDNN autotune observed on first call (HANDOFF §5.1).

## Alternatives considered

- **Two services (the sidecar plan)** — pays the CUDA/cuDNN overhead twice,
  adds an IPC hop and a second failure domain, and requires a GPU memory
  split negotiated between processes. Explicitly abandoned and deleted.
- **Unload/reload on switch** — one framework resident at a time; switching
  costs a multi-second checkpoint load plus autotune. Rejected in favour of
  both-resident (FR-064).
- **GPU partitioning (MIG/MPS)** — not available on consumer Turing cards.

## Decision

One Python process hosts the gRPC server, the PyTorch engine, and the
TensorFlow engine; all engines load at startup and stay resident; all GPU
work is serialised through one process-wide asyncio lock (FR-023).

## Consequences

Positive:
- CUDA context and cuDNN overhead paid once — the budget that makes
  NFR-PERF-010's both-resident target (≤ 5 632 MiB on 6 144 MiB) plausible.
- Switch cost is a stream restart, not a checkpoint reload (FR-064,
  NFR-REL-006).
- Per-engine load-failure tolerance gives degraded service on TF-less hosts
  (FR-024, NFR-COMPAT-003, NFR-FLEX-001).

Negative:
- Single failure domain: a crash in either framework's native code kills
  every mode at once.
- Version coupling: both frameworks must coexist in one interpreter — the
  reason the full server is WSL2-only (GPU TensorFlow has no native Windows
  build since 2.11) and E-D can serve only the PyTorch modes.
- Loop contention risk: heavyweight framework calls in one process make
  event-loop lag the collapse-predictor metric (NFR-PERF-006 exists to
  watch exactly this).
- The combined resident footprint has **never been measured** — TR calls it
  the single largest unquantified risk; NFR-PERF-010 is the target that
  forces the measurement.

## Requirements served

NFR-PERF-010, FR-023, FR-024, FR-064; NFR-REL-006, NFR-COMPAT-003,
NFR-FLEX-001. Costs land on: NFR-PERF-006, NFR-PERF-007, E-M dependence.

## Evidence

TR §4.5, §9.4 (Table 1 item 8); HANDOFF §3.3, §5.1–5.2, §9.5;
`src/grpc_server.py:36-38` (lock), `src/model_registry.py:202-213`
(load tolerance).
