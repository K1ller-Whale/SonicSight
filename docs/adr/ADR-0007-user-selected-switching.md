# ADR-0007 — User-selected model switching rather than automatic fusion or gating

**Status:** Accepted (as built; recorded retrospectively 2026-08-08).
Supersedes the abandoned fusion design (`gate_masks`, `CAM_GATE_STRENGTH`),
which was deleted before implementation (HANDOFF §9.5).

## Context

An earlier design proposed fusing the two models — the multisensory model's
class activation map gating the Sound of Pixels masks. Three facts killed
it. The models' domains are disjoint (musical instruments versus speech), so
no principled arbiter exists for a mixed scene — a gate would blend a valid
answer with an out-of-domain one. Their capture profiles are incompatible
(8 fps/11 025 Hz halves versus 30 fps/22 050 Hz letterboxed full frames), so
one phone stream cannot feed both simultaneously without capturing at the
union of both profiles. And running both per window would double GPU work on
a card where the speech branch alone runs a 57–59 % duty cycle at its hop.

## Alternatives considered

- **CAM-gated fusion** (the abandoned plan) — no validity story for
  cross-domain blending; doubles inference; deleted.
- **Output ensembling/blending** — same objections, plus the two models'
  outputs mean different things (spatial halves versus on/off-screen).
- **Automatic scene classification choosing the model** — adds an
  unvalidated third model whose failure mode (wrong mode, silently) is
  worse than the user's occasional wrong pick; nothing in either paper
  supports it.

## Decision

The user picks one model; that model alone produces both audio and map.
Selection is per stream via metadata; switching is cancel-and-reopen with a
700 ms settle; every result echoes its model id and the client drops
mismatches; both engines stay resident (ADR-0004) so no reload cost.

## Consequences

Positive:
- Each mode's output is exactly what its model was validated to produce —
  no blended artefact with no evidence base (O3; the §2.2.2 honesty
  requirement).
- Track labels can be mode-truthful ("Left/Right" versus
  "On-screen/Off-screen") because modes never mix (NFR-INT-002).
- Switch correctness is testable as a protocol property (FR-062, FR-063,
  NFR-REL-006) rather than a model-quality question.

Negative:
- The burden of choosing correctly lands on the user: a speech scene in
  music mode simply performs badly, and nothing detects or flags it (no
  competence estimate exists outside the speech gate — TR §11.1).
- A switch costs a stream teardown, 700 ms settle, and a fresh window fill —
  ≈ 6.6 s of dead time on the halves branch (0.7 s settle + 5.9 s fill;
  NFR-PERF-001's bound is the fill this incurs).
- In-flight stale results require the model-id echo-and-drop machinery —
  protocol surface that exists only because of switching.

## Requirements served

FR-062, FR-063, FR-064; NFR-REL-006, NFR-INT-002; O2, O3.
Costs land on: UC-02 (user burden), NFR-PERF-001 (refill after switch).

## Evidence

HANDOFF §3.1–3.2, §9.5; TR §3.4–3.5 (Table 4), §11.1; `MainActivity.kt:191-202`;
`MainViewModel.kt:275-278`; `src/grpc_server.py:52-67`.
