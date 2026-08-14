# Phase 5 result validation — audit of the committed p5-* runs (2026-08-10)

Scope: the nine `SonicSightBackend/loadtest/results/p5-*` runs committed in
`59adf8a` (executed 2026-08-08/09 on DESKTOP-RF4V9RH, Windows 10 26200,
GTX 1650 4 GiB, Python 3.11.9, torch 2.6.0+cu124, no TensorFlow). Audit
performed by a 23-agent adversarial review (three independent auditors,
per-claim verification passes) plus manual cross-reading. Verdicts below are
the *confirmed* set; refuted claims are recorded at the end.

## A. Findings that limit what the p5 results may claim

1. **Targets-file drift mid-campaign (invalidates assertion outcomes of six
   runs).** Six runs (smoke, baseline, baseline-pixel, baseline-pixel-v2,
   spike, load) record `nfr_targets.yaml` sha256 `b43885f80c4f1c08`
   (25497 B); three (stress, baseline-v2, switching) record
   `afe89a851eb00392` (25614 B). The edit landed between p5-load's end
   (2026-08-08T15:32:39Z) and p5-stress's end (15:40:57Z). NFR-PERF-007 was
   therefore judged under two different targets versions. Neither file was
   ever committed (docs/ sits outside all three repositories), so the six
   old-yaml assertion outcomes cannot be re-derived from this workspace.
   **Measured metric values remain valid; assertion verdicts of the six
   old-yaml runs are provisional until re-evaluated against a recovered or
   reconstructed final yaml.**

2. **p5-stress's "concurrency ceiling = 1" is an artifact, not capacity.**
   Only step n=1 ran (wall time = exactly one 420 s step). The step passed
   both p95 gates (server-time p95 89 ms ≤ 140; cadence dev p95 60.6 ≤
   62.5) and failed only PERF-004's max ≤ 500 ms, via two isolated slow
   cycles (1338 ms, 1476 ms) on a server that had been up ~70 min and had
   just been driven ~123 s behind real time by the load scenario 77 s
   earlier. Baseline-v2 on a fresh server the next morning passed with
   steady max 328 ms. The stress scenario was never rerun on a fresh
   server. **Do not cite first_breaching_concurrency=1 as the system's
   concurrency ceiling.**

3. **All nine runs record a dirty backend tree.** Head `1a1842a8` +
   uncommitted changes. The delta was later committed as `59adf8a` and
   touches only `loadtest/*` and new `tests/*` — the server under test was
   the recorded commit; the *harness* was ahead of it. Provable: the v2
   runs emit a `total_server_time_ms p99` field that `1a1842a8`'s
   aggregator cannot produce. Disclose in the report: results reproduce
   from `59adf8a`, not from the recorded head.

4. **Two server processes served the campaign, not one.** Aug 8 runs:
   pid 26124 (up 14:24:45Z); Aug 9 runs: pid 408 (up 11:31:50Z). The Aug 8
   log ends mid-run (an aborted switching-style run discarded without
   record). Cross-run resource comparisons must not span the restart.

## B. Genuine performance findings (reproducible, report-worthy)

5. **NFR-PERF-008 FAIL — pixel query round trip.** p95 = 944.8 ms (v1) and
   1182.9 ms (v2) vs the 250 ms target; p99 up to 3064 ms. Reproduced
   across both pixel baselines including after the v1→v2 timer-precision
   fix (v1's RTTs were quantized to 0 at p50 — Windows coarse timer; v2
   used a high-resolution counter). The only measurable context for
   PERF-008, and it fails in both. A real defect or a target needing
   reconciliation with a recorded reason — not noise.

6. **NFR-PERF-007 FAIL on the GTX 1650 host (genuine for that host).**
   2 concurrent sessions, 720 s: 0 of 2 sessions met PERF-003/004;
   server-time p95 310.75 ms, max 4475 ms; audio-age lag climbed to
   ~123 s. The 4 GiB GTX 1650 cannot sustain two halves sessions. (Judged
   under the old yaml — finding 1 — but the measured values alone carry
   the conclusion.)

7. **NFR-PERF-014 FAIL (spike).** All 4 burst sessions survived, but none
   reached a compliant 10 s cadence window in the 120 s spike run
   (measured = inf). Note the client-contention caveat (finding 12).

8. **NFR-REL-006 FAIL (switching) on GPU delta only.** GPU +250 MiB vs
   baseline after 100 cycles (target ≤ 64), RSS −26.8 % (pass). Device-
   global nvidia-smi reading; single-sample baseline/final points; still a
   plausible allocator-growth signal worth re-measuring on E-M.

9. **NFR-MAINT-001 FAIL — 2 unit-test collection errors** in
   `tests/test_grpc_client.py`: it is a manual live-server script whose
   `test_*` functions take a `stub` parameter; pytest misreads it as a
   fixture. Reproduced identically on the E-M host. Addressed 2026-08-10
   by `tests/conftest.py` `collect_ignore` (classification fix, not a
   hidden failure).

## C. Suite-measurement limitations (interpretation guards)

10. **sequence_gap_rate (PERF-013) is structurally near-vacuous** — gaps
    are counted only across strictly increasing sequence pairs, so most
    loss modes cannot produce a FAIL. Its passes are weak evidence.
11. **The asserted "max" is a steady-window max** that excludes warmup
    outliers differently per scenario (baseline-v2's raw distribution has
    a 2217 ms warmup cycle its assertion never sees). Methodology is
    consistent within a scenario; do not compare raw characterisation max
    against assertion max across scenarios.
12. **Client-side contention:** N driver sessions share one asyncio loop
    and synchronous JPEG encodes; under load/stress/spike some cadence
    misses can originate in the driver, biasing toward FAIL
    (conservative direction, but say so when citing).
13. **NFR-SEC-001's count==0 is weak evidence:** endpoint sampling every
    ~30 s misses transient connections.
14. **Cadence metric is named `…from_125ms` but measures deviation from
    the per-model hop** (250 ms for multisensory). Naming debt, not a
    math error.
15. **condition-mismatch outcomes do not affect the exit code**, so a
    mis-targeted run can exit 0; PERF-009 has no committed evidence in
    any run (condition-mismatch everywhere applicable).
16. Minor: spike serializes `Infinity` (non-strict JSON); soak's RSS gate
    keys on requested rather than achieved duration; loop-lag p99 is
    diluted by idle spans (max unaffected); local-time log stamps vs UTC
    metadata (3 h offset on the p5 host); multi-MB raw captures
    (current-20260808/server.log ~8 MB, loop_lag.csv ~86 k lines) were
    committed against the working rule — summarise-and-prune candidates.

## D. Refuted claims (checked, not real)

- "RSS monitor attaches to different processes across runs" — refuted;
  attachment is consistent per campaign.
- "Loop-lag sampler missed a 12 s event-loop stall on Aug 8" — refuted;
  the gap in the CSV corresponds to process restart, not a missed stall.

## E. Consequences applied on 2026-08-10 (E-M host)

- `docs/nfr/nfr_targets.yaml` reconstructed (provenance-annotated;
  final-yaml values byte-faithful where printed in summaries; E-M targets
  newly set pre-measurement; sha will differ from `afe89a851eb00392` —
  every new run's metadata records which file it used).
- The E-M campaign (GTX 1660 Ti 6 GiB, WSL2, TF 2.21 + torch 2.11) reruns
  every scenario on a single fresh server, adds multisensory coverage
  (PERF-002/005/010, FUNC-002, FLEX-001 E-M half, cross-framework
  switching), and re-tests stress on a non-saturated server to replace
  finding 2's artifact with a defensible ceiling.
