# ADR-0006 — Sliding window with overlap-add rather than independent chunk inference

**Status:** Accepted (as built; recorded retrospectively 2026-08-08)

## Context

Both models are windowed by construction: Sound of Pixels consumes 65 536
samples (5.944 s) and the multisensory model 2.135 s nominal (the locked
`vid_dur` parameter; ≈ 2.102 s of audio at the wire rate) — nothing shorter
is a valid input, and the windows are checkpoint-locked (TR §8.2). A listener,
however, needs results at interactive cadence, and consecutive windows
processed independently would disagree at their boundaries, producing
audible seams.

## Alternatives considered

- **Independent chunk inference (chunk = window)** — one result per 5.9 s /
  2.1 s: unusable cadence, discontinuous output at every boundary. Rejected.
- **Independent chunks with short windows** — invalid: the record shows a
  changed window produces confidently wrong output (the 4.27 s FAIL,
  TR §8.2). Not an option at all.
- **Snapshot-on-demand (infer only when the user asks)** — abandons the live
  overlay and continuous playback that define the system (O1). Rejected.

## Decision

Advance a full-length analysis window by a small hop (1 378 samples =
125 ms halves/touch; 5 512 samples = 250 ms speech), infer per hop, and
stitch outputs through a timeline-driven overlap-add buffer: absolute
next-sample accounting, a held-back crossfade tail re-read from the next
window (sin²/cos² fades), and counters for gap fills, splice fallbacks, and
lagging windows.

## Consequences

Positive:
- Result cadence is decoupled from window length: 8 results/s from a 5.9 s
  window (NFR-PERF-003); the overlay updates at 8/4 Hz.
- Output is gapless and duplicate-free by construction, and violations are
  observable via the counters (FR-032, NFR-REL-005).
- The hop is a per-model constant, so the speech branch's 250 ms hop —
  forced by its 142–148 ms inference time — is a registry entry, not a
  special case (ADR-0005).

Negative:
- Compute multiplication: every output second is inferred ≈ 47.6× on the
  halves branch (65 536/1 378) and ≈ 8.4× on the speech branch — the GPU
  duty-cycle bill on speech (57–59 % at its hop, derived from the measured
  142.6–148 ms inference time) that caps concurrency (NFR-PERF-007).
- The latency floor: ≈ half a window of unavoidable lookahead is the
  dominant term in the §2.2.3 perceived-lag estimates. This decision
  inherits the models' constraint; it does not create it, but it cannot
  remove it.
- Mechanism complexity: the held-back tail and splice-fallback path are
  real failure modes (counted, and thresholded in NFR-REL-005).

## Requirements served

FR-021, FR-032, FR-033; NFR-PERF-003, NFR-REL-005; O1. Costs land on:
NFR-PERF-007, the §2.2.3 latency finding.

## Evidence

`src/overlap_add_buffer.py` (timeline contract :10-27, crossfade :100-115,
counters :49-52); `src/model_registry.py:108,136` (hops); TR §7.4, §8.2;
HANDOFF §5.3.
