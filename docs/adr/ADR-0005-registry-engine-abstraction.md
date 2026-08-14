# ADR-0005 — Registry plus engine interface rather than conditional branching per model

**Status:** Accepted (as built; recorded retrospectively 2026-08-08)

## Context

Three modes share one stream loop, but every per-stream object — buffer
length, sample rates, frame-selection rule, ring cap, hop, heatmap
convention, output labels — differs per model. Encoding that variation as
`if model == ...` branches through the server would couple every model to
every layer and make each addition a cross-cutting edit. The project also
has hard evidence that per-model constants must not be tunable: deviating
from the locked window length and map-resolution flag each produced
confident, plausible-looking, **wrong** output (TR §8.2).

## Alternatives considered

- **Conditional branching per model in the stream loop** — N models × M
  touch points; every addition edits the core loop; constants scattered.
  Rejected.
- **Config-file / environment-variable model parameters** — invites exactly
  the "reasonable" deviations the record shows produce silent wrongness.
  Rejected deliberately (the specs are frozen dataclasses, not config).
- **Inheritance hierarchy without a registry** — still needs an id→class
  map somewhere; the registry *is* that map, made explicit and testable.

## Decision

A registry of frozen `ModelSpec` dataclasses resolved by metadata id, each
carrying every constant the stream shape depends on plus an engine factory;
engines implement a common adapter contract (framework imports inside
`load()`, resampling internal, `{left, right, heatmaps, diag}` return);
the server reads the spec and branches once, on `spec.mode`.

## Consequences

Positive:
- Two historical additions (multisensory, pixel) required zero changes to
  buffering, overlap-add, or transport — the NFR-MAINT-002 claim is this
  decision working.
- Constants live in one frozen, unit-tested place (FR-060, FR-066,
  NFR-MAINT-003; 13 registry tests).
- Load tolerance and degraded service fall out of the engine contract
  (FR-024, NFR-COMPAT-003).

Negative:
- Indirection: reading one request's behaviour means following
  spec → factory → engine singleton → inference module.
- Frozen specs make every capture-geometry variation a code change and
  release — inflexibility is the point, but it is still a cost.
- Engines are process-wide singletons, so cross-*session* state that lives
  on an engine (the normalisation EMAs) is shared between concurrent
  streams — the FR-020 **partially satisfied** finding is a direct cost of
  this choice.

## Requirements served

FR-060–FR-066, FR-024; NFR-MAINT-002, NFR-MAINT-003, NFR-COMPAT-003; O2, O5.
Costs land on: FR-020 (PS).

## Evidence

`src/model_registry.py` (frozen dataclass :28, registry :185-189, loader
:202-213); `src/engines/*.py`; MODELS.md; TR §4.3, §8.2;
`tests/test_model_registry.py`.
