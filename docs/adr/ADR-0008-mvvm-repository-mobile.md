# ADR-0008 — MVVM with a Repository layer on the mobile side

**Status:** Accepted (as built; recorded retrospectively 2026-08-08) — with
the deviations from the pattern recorded below as facts, not aspirations.

## Context

The client is a single-screen, media-heavy app: camera and microphone
capture, JPEG encoding, a bidirectional gRPC stream, dual-track playback
with jitter buffering, and an overlay pipeline. It needs per-mode
configuration (three profiles), staleness filtering on a switchable stream,
and enough separation that contract logic is JVM-unit-testable without a
device.

## Alternatives considered

- **Activity-owns-everything** — no testable seam anywhere; rejected as a
  baseline, though (see below) the media pipeline in practice still lives
  in the Activity.
- **MVI / single-state Redux-style store** — stricter unidirectional data
  flow and replayable state, at the cost of ceremony poorly matched to
  continuous binary streams (PCM chunks through a reducer is the wrong
  tool). Not adopted.
- **Jetpack Compose + ViewModel** — a view-layer rewrite orthogonal to the
  problems this app has; the XML View layer predates the decision point.
  Not adopted.
- **Full Clean Architecture (use-case/interactor layers)** — layering
  overhead unjustified for one screen and one data source.

## Decision

MVVM: `MainViewModel` (AndroidViewModel) holds stream lifecycle, profile
selection, staleness filtering, and result fan-out as
LiveData + SharedFlows; `GrpcVideoRepository` wraps the generated stub and
exposes the stream as a cold Kotlin Flow; `GrpcModule` owns the channel and
host preference. No dependency-injection framework; the repository is
constructed directly.

## Consequences

Positive:
- The wire-facing contract logic — model-id filtering (FR-063), profile
  mirroring (FR-065), sequence-gap handling (FR-052) — lives in the
  ViewModel/Repository seam, where JVM tests can reach it; the
  coordinate-mapping tests (FR-045) exist because that seam exists.
- Stream restarts on switch are a ViewModel concern with a clean
  fresh-flow-per-stream design (ADR-0007's client half).

Negative (where the pattern factually breaks, per the working tree):
- The media pipeline — CameraX analysis, JPEG encode, `AudioRecord` loop,
  `AudioTrack`/jitter-buffer lifecycle, proto chunk construction, error-
  string mapping — lives in `MainActivity` (~1 000+ lines), coupled to the
  Activity lifecycle and untestable off-device (NFR-INT-001 needs an
  instrumented test for exactly this reason).
- Unidirectional data flow is bypassed in places: `@Volatile` mutable
  fields on the ViewModel (`frozen`, `gridDims`) are read directly by the
  Activity, outside observation.
- No DI means substituting the repository in tests is awkward; the
  singleton channel module carries hidden state (host preference).
- These deviations are recorded facts of the design, and the §8 rationale
  reports them as costs — not as future work dressed as architecture.

## Requirements served

FR-045, FR-052, FR-063, FR-065; NFR-INT-002; O5. Costs land on:
NFR-INT-001 (instrumented-only verification), FR-006 (lifecycle handling
absent — the NS finding sits in the layer the pattern left in the
Activity).

## Evidence

`MainViewModel.kt:44-183, 227-278, 304-313`; `GrpcVideoRepository.kt:15-60`;
`GrpcModule.kt:9-68`; `MainActivity.kt:888-1114` (media pipeline in the
Activity), `:79-81/118-120` referenced fields; TR §5.1, §5.7;
SonicSightMobile `README.md:41`.
