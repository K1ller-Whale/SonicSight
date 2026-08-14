# ADR-0001 — Client–server inference rather than on-device

**Status:** Accepted (as built; recorded retrospectively 2026-08-08, from the
working trees and the project measurement record)

## Context

Both models are pretrained research artefacts: Sound of Pixels is three
PyTorch checkpoints totalling ≈ 158.8 MiB; the multisensory model is a
TensorFlow-1-era graph with a ≈ 1.27 GiB checkpoint that runs under
`tf.compat.v1` on TensorFlow 2.21. The multisensory forward pass measures
142.6 ms median per 2.1 s window on a GTX 1660 Ti and 1991 ms on a desktop
CPU (TR Table 13) — a 14× spread that no phone-class processor closes.
Neither model has a mobile runtime port: the PyTorch port of the multisensory
model is a skeleton with no checkpoint (TR Table 1, item 12), and no
TFLite/ONNX conversion of either has been attempted or validated. Continuous
camera-plus-network capture is already the phone's thermal budget; adding
sustained neural inference is not plausible on the target device class.

## Alternatives considered

- **On-device inference (TFLite/ONNX/ExecuTorch port + quantisation)** — no
  validated port of either model exists; porting and quantisation risk to
  model validity is exactly the class of risk the locked-constants record
  (TR §8.2) exists to avoid. Rejected.
- **Hybrid split (features on device, synthesis on server)** — still places
  sustained neural compute on the phone and doubles the contract surface.
  Rejected.
- **Dedicated edge box beside the phone** — re-creates the server with
  worse logistics; serves no requirement. Rejected.

## Decision

The Android client captures, transmits, plays back, and renders overlays;
all neural inference runs on a LAN server with a CUDA GPU hosting both
frameworks.

## Consequences

Positive:
- Server-grade cycle times are attainable (NFR-PERF-004/005 targets exist
  because the GPU record supports them).
- Both frameworks and full-size checkpoints run unmodified — no porting or
  quantisation risk to either model's validity (O2, O3).
- The phone's budget is spent on capture and playback only.

Negative:
- Network dependency: no session without the server (UC-07); LAN conditions
  become part of the reference environment (E-N).
- A transport latency term is added to an already window-dominated lag
  budget (§2.2.3).
- Captured audio and video leave the device in plaintext — the S5 bystander
  exposure and the NFR-SEC-001/002 posture exist because of this decision.
- One GPU serialises all users (FR-023): the concurrency ceiling
  NFR-PERF-007 will measure is a structural consequence, and product-scale
  service is out of scope (FR-P08).
- On-device/offline operation is foreclosed (FR-P03).

## Requirements served

O1, O2; NFR-PERF-004, NFR-PERF-005, NFR-FLEX-001. Costs land on: UC-07,
NFR-SEC-001/002, NFR-PERF-007, FR-P03, FR-P08.

## Evidence

TR §3.3, §9.3 (Table 13), Table 1 items 3–4 and 12; checkpoint sizes TR §9.4
(verified on disk 2026-08-08, ANALYSIS_REPORT Appendix D); HANDOFF §5.1.
